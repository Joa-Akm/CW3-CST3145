<template>
  <header>
            <div class="container">
               <div class="row">
                <!-- Logo and Mobile Menu -->
                  <div class="col-md-3 col-sm-3 col-lg-3 col-xl-3 col-3">
                     <div class="logo">
                        <a href="index.html" title="logo"> <img src="images/logo.png" width="80" height="80" alt="" srcset=""> </a>
                     </div>
                     <div class="mob-menu">
                        <span>
                           <i class="fa fa-bars"></i>
                        </span>
                     </div>
                  </div>
                  <!-- Cart Button -->
                  <div class="col-md-9 col-sm-9 col-lg-9 col-xl-9 col-9">
                    <div style="float: right; margin-top: 1.5em;">
                        <div class="container">
                            <button class="btn btn-primary mb-4 mt-6" v-on:click='showCheckout' :disabled="cart.length === 0">
                               
                           <i class="fa-solid fa-cart-plus"></i>
                           <span class="badge rounded-pill badge-notification bg-danger">{{cartItemCount}}</span>
                       </button>
                       </div>
                    </div>
                  </div>
               </div>
            </div>
         </header>
         <div class="container-fluid mt-4">
            <div v-if="showCourses">
                <Lesson />
            </div>
            <!-- Checkout Section -->
            <div v-else>
                <Checkout />
            </div>     
          </div>
</template>

<script>
import Checkout from './components/Checkout.vue'
import Lesson from './components/Lesson.vue'
export default {
  name: 'App',
  components: {
    Checkout,
    Lesson
  },
  // Fetching data from server on component creation
  created:()=>{
    fetch(`http://localhost:3000/collection/lessons`).then(res => res.json())
        .then(res => {
          this.lessons = res
        })
  },
  // Vue.js Data
  data(){
    return {
    sitename:'Lessons Activities',
    lessons: [],  // Array to hold fetched data
    showCourses: true,
    cart: [],   // Array to hold cart items
    name: '',
    phone: '',
    sortOrder: 'Ascending',
    searchInput: '',
    filterType:'--Sort By--',
    nameError: '',
    phoneError: '',
  }
  },
  // Vue.js Methods
  methods: {
    addToCart(lessons) {
      lessons.availableSpaces--
      const id = Math.ceil(Math.random()*10000)+`${lessons.title}`
      this.cart.push({id:id, "lessons":lessons})

    },
    // Check if item can be added to cart
    canAddToCart: function(lessons) {
      return lessons.availableSpaces > this.cartCount(lessons.id);
    },
    removeCartItem(cart_item) {
      for (let i = 0; i < this.data.length; i++) {
        const cuse = this.data[i];

        if (cart_item.lessons.id == cuse.id) {
          cuse.availableSpaces++
          break;
        }
      }
      for (let i = 0; i < this.cart.length; i++) {
        const item = this.cart[i];

        if (cart_item.id == item.id) {
          this.cart.splice(i,1)
          break;
        }
      }

    },
    // Count occurrences of item in cart
    cartCount: function(id) {
      let count = 0;
      for(let i = 0; i < this.cart.length; i++)
      {
        if(this.cart[i] === id)
        {
          count++;
        }
      }
      return count;

    },
    showCheckout() {
      this.showCourses = this.showCourses ? false : true;
    },
    checkOut() {
      let show = this.cart
      return show
    },
    validateName() {
      if (this.name.trim() === '') {
        this.nameError = 'Name is required'
      } else {
        this.nameError = ''
      }
    },
    validatePhone() {
      const phoneRegex = /^[0-9]{10}$/
      if (!phoneRegex.test(this.phone.trim())) {
        this.phoneError = 'Phone number must be 10 digits'
      } else {
        this.phoneError = ''
      }
    },
    submitForm(){
      alert('Request submitted');
      let data = {}
      for (let i = 0; i < this.cart.length; i++) {
        const item = this.cart[i].lessons;
        if (!data[`${item["_id"]}`]) {
          data[`${item["_id"]}`] = {
            "availableSpaces":1,
            "item":item
          }
        }else{
          data[`${item["_id"]}`]["availableSpaces"]++
        }
      }
      console.log(data);

      const keys = Object.keys(data)
      const values = Object.values(data)
      fetch(`http://localhost:3000/collection/orders`,{
        method:'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          "name":this.name,
          "phone":this.phone,
        })
      })
          .then(result => result.json())
          .then(result =>{
            if (result.length == 1) {
              fetch(`http://localhost:3000/collection/orders/${result[0]["_id"]}`,{
                method:'PUT',
                headers: {
                  'Content-Type': 'application/json'
                },
                body: JSON.stringify({"items":values})
              }).then(result => result.json())
                  .then(result => {
                    for (let i = 0; i < keys.length; i++) {
                      const key = keys[i];
                      const value = values[i];
                      fetch(`http://localhost:3000/collection/lessons/${key}`,{
                        method:'PUT',
                        headers: {
                          'Content-Type': 'application/json'
                        },
                        body: JSON.stringify({"availableSpaces":value["item"].availableSpaces})
                      })
                    }
                    this.cart = []
                  })
            }
          }).catch(err =>{
        console.log(err);
      });
    },

    customSearch(){

      fetch(`http://localhost:3000/search?keyword=${this.searchInput}`)
          .then(res => {
            return res.json()
          })
          .then(data => {
            this.data = data
          })
          .catch(err => {
            this.data = []
            console.log(`unable to get lessons: ${err}`)
          })
    },

    sortPrice: function(priceArray){
      function compare(a,b) {
        if (a.price > b.price)
          return 1;
        if (a.price < b.price)
          return -1;
        return 0;
      }
      return priceArray.sort(compare);
    },

    sortLocation: function(locationArray){
      function compare(a,b) {
        if (a.location > b.location)
          return 1;
        if (a.location < b.location)
          return -1;
        return 0;
      }
      return locationArray.sort(compare);

    },
    sortTitle: function(titleArray){
      function compare(a,b) {
        if (a.title > b.title)
          return 1;
        if (a.title < b.title)
          return -1;
        return 0;
      }
      return titleArray.sort(compare);

    },
    // Sort array by available spaces
    sortSpaces: function(spacesArray){
      function compare(a,b) {
        if (a.availableSpaces > b.availableSpaces)
          return 1;
        if (a.availableSpaces < b.availableSpaces)
          return -1;
        return 0;
      }
      return spacesArray.sort(compare);
    },
    // Filter sessions based on sort and search criteria
    filterSessions: function() {
      let lessonses = this.lessons;
      if (this.filterType === 'price') {
        lessonses = this.sortPrice(lessonses)
      }
      else if (this.filterType ==='title') {
        lessonses = this.sortTitle(lessonses)
      }
      else if (this.filterType ==='location') {
        lessonses = this.sortLocation(lessonses)
      }
      else if (this.filterType ==='availableSpaces') {
        lessonses = this.sortSpaces(lessonses)
      }

      if (this.sortOrder == 'Ascending'){
        return lessonses
      }
      else if (this.sortOrder =='Descending'){
        return lessonses.reverse()
      }
      return lessonses

    }
  },
  // Vue.js Computed Properties
  computed: {
    cartItemCount: function()
    {
      return this.cart.length;
    },
    isValid() {
      return this.name.trim() !== '' && this.phone.trim() !== ''
    }
  },
}
</script>

<style>

</style>
