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
                <Lesson v-model:sortOrder="sortOrder" v-model:sortBy="filterType"  :custom-search="customSearch" :lessons="lessons" v-on:add-to-cart="addToCart" :can-add-to-cart="canAddToCart" :cart-item-count="cartItemCount"/>
            </div>
            <!-- Checkout Section -->
            <div v-else>
                <Checkout :submit-form="submitForm" :cart="cart" v-on:remove-item-from-cart="removeCartItem" />
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
  // Vue.js Data
  data(){
    return {
    lessons: [],  // Array to hold fetched data
    showCourses: true,
    cart: [],   // Array to hold cart items
    name: '',
    phone: '',
    sortOrder: 'Ascending',
    searchInput: '',
    filterType:'--Sort By--',
  }
  },
  // Fetching data from server on component creation
  created(){
    fetch(`http://localhost:3000/collection/lessons`).then(res => res.json())
        .then(res => {
          this.lessons = res
        })
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
      for (let i = 0; i < this.lessons.length; i++) {
        const cuse = this.lessons[i];

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

    submitForm(name, phone){
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
          "name":name,
          "phone":phone,
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
                    console.log(result)
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

    customSearch(search){

      fetch(`http://localhost:3000/search?keyword=${search}`)
          .then(res => {
            return res.json()
          })
          .then(data => {
            this.lessons = data
          })
          .catch(err => {
            this.lessons = []
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
    }
  },
}
</script>

<style>
@import url(http://fonts.googleapis.com/css?family=Roboto:400,300);

.card .card-image{
  overflow: hidden;
  -webkit-transform-style: preserve-3d;
  -moz-transform-style: preserve-3d;
  -ms-transform-style: preserve-3d;
  -o-transform-style: preserve-3d;
  transform-style: preserve-3d;
}

.card .card-image img{
  -webkit-transition: all 0.4s ease-in-out;
  -moz-transition: all 0.4s ease-in-out;
  -ms-transition: all 0.4s ease-in-out;
  -o-transition: all 0.4s ease-in-out;
  transition: all 0.4s ease-in-out;
}

.card .card-image:hover img{
  -webkit-transform: scale(1.2) rotate(-7deg);
  -moz-transform: scale(1.2) rotate(-7deg);
  -ms-transform: scale(1.2) rotate(-7deg);
  -o-transform: scale(1.2) rotate(-7deg);
  transform: scale(1.2) rotate(-7deg);
}

.card{
  font-family: 'Roboto', sans-serif;
  border:none;
  margin-top: 10px;
  position: relative;
  -webkit-box-shadow: 0 2px 5px 0 rgba(0, 0, 0, 0.16), 0 2px 10px 0 rgba(0, 0, 0, 0.12);
  -moz-box-shadow: 0 2px 5px 0 rgba(0, 0, 0, 0.16), 0 2px 10px 0 rgba(0, 0, 0, 0.12);
  box-shadow: 4 2px 5px 0 rgba(0, 0, 0, 0.16), 0 2px 10px 0 rgba(0, 0, 0, 0.12);
}

.card .card-content {
  padding: 10px;
  background:#1A9AE1;
  color:white;
}

.card .card-content .card-title, .card-reveal .card-title{
  font-size: 24px;
  font-weight: 200;
}

.card .card-reveal{
  padding: 20px;
  position: absolute;
  background-color: #FFF;
  width: 100%;
  overflow-y: auto;
  left:0;
  bottom:0;
  height: 100%;
  z-index: 1;
  display: none;
}

.card .card-reveal .card-title{
  color: rgba(0, 0, 0, 0.71);
  margin:10px;
  font-size:2.2rem;
}

.card .card-reveal p{
  color: rgba(0, 0, 0, 0.71);
  margin:10px;
  font-size:1.2rem;
}

.btn-custom{
  background-color: transparent;
  font-size:18px;

}

header {
  transition: all 0.3s ease-in-out;
  box-shadow: 0 1px 6px 0 rgba(214, 220, 239, 1);
  top: 0px;
  background-color: #fff;
  position: sticky;
  z-index: 40;
  height: 100px;
}
.logo {
  padding:5px 0px;
}
.logo a {
  color: #3a76cb;
  text-transform: capitalize;
  color: #000000;
  font-size: 35px;
  font-weight: 500;
  text-decoration: none;
}
.mob-menu {
  display: none;
}
.main-menu {
  margin: 20px 0px;
  float: right;
}
.main-menu ul.nav {
  float: left;
  margin-right: 20px;
  list-style: none;
  padding-left: 0px;
  margin-bottom: 0px;
  padding-top:10px;
}
.nav li:first-child {
  margin-left: 0px;
}
.main-menu ul {  display: inline-block; }
.main-menu ul li {
  position: relative;
  display: inline-block;
  margin: 12px 20px;
}
.main-menu ul li a {
  font-size: 20px;
  font-weight: 500;
  line-height: 1.3;
  color: #3a76cb;
  display: block;
  text-decoration:none;
}
.main-menu ul.right-nav{
  padding-left:0px;
}
.main-menu ul.right-nav li a {
  width: 150px;
  height: 45px;
  border-radius: 4px;
  background-color: #fff;
  text-align: center;
  color: #3a76cb;
  border: solid 1px #3a76cb;
  vertical-align: middle;
  display: table-cell;
}
.main-menu ul.right-nav li:last-child {
  margin-right: 0px;
}
.main-menu ul.right-nav li.active a {
  background-color: #3a76cb;
  color: #fff;
}
@media(max-width:1000px){
  .logo {
    display: inline-block;
    width: 78%;
    padding: 10px 10px 10px 0px;
  }
  .mob-menu {
    display: none;
    width: 20%;
  }
  .mob-menu span {
    border: solid 2px #3a76cb;
    display: block;
    text-align: center;
    border-radius: 4px;
    padding: 2px 6px;
    width: 50px;
    color: #3a76cb;
    font-size: 25px;
    cursor: pointer;
  }
  .main-menu {
    margin: 0px 0px 10px 0px;
    float: left;
    background-color: #f1f1f1;
    width: 100%;
    display: none;
  }
  .main-menu ul {
    list-style: none;
    padding-left: 0px;
    margin-bottom: 0px;
    width: 100%;
  }
  .main-menu ul.right-nav {
    text-align: center;
    margin-top: 6px;
  }
  ul.right-nav {
    margin-bottom: 10px;
  }
  .main-menu {
    margin: 0px 0px 10px 0px;
    float: left;
    background-color: #f1f1f1;
    width: 100%;
    display: none;
  }
  .main-menu ul li {
    display: block;
    width: 100%;
    margin: 0px;
  }
  .nav li:first-child {
    margin-left: 0px;
  }
  .main-menu ul li a {
    padding: 10px 15px;
    display: block;
    border-bottom: solid 1px #e2e4e6;
    text-align: center;
  }
  .main-menu ul.right-nav li {
    margin-left: 0px;
    margin-right: 30px;
    display: inline-block;
    width: 150px;
  }
  .main-menu ul.right-nav li a {
    font-size: 18px !important;
  }
}
#search_parent {
  width: 300px;
  height: 40px;
  border: 1px solid gray;
  margin: 50px auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: white;
  border-radius: 5px;
}
@media(max-width:768px){
  #search_parent {
    margin: 30px auto;
  }
  .text{
    text-align: center;

  }
  .p-5
  {
    padding: 0;
  }
}

#search_parent>* {
  height: 97%;
}

#search_parent>input {
  width: 250px;
  outline: none;
  border: none;
  padding-left: 10px;
  border-radius: 5px 0 0 5px;
  font-size: 18px;
}

#search_parent>label {
  width: 45px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 25px;
  cursor: pointer;
}
.wrapper:hover .box:not(:hover)  {
  filter: blur(3px);
  opacity: 0.5;
}
</style>
