<template>
  <!-- Search and Sorting Section -->
                <div class="row" style="width:100%">
                    <!-- Title and Search Input -->
                    <div class="col-md-7 col-sm-12 col-12">
                        <h1 v-text="sitename" class="p-5 text"></h1>
                    </div>
                    <!-- Sort By and Sort Order -->
                    <div class="col-md-4 col-sm-12 col-12">
                        <div id="search_parent">
                            <input type="text" @input="customSearch" id="search1" placeholder="Search here..." v-model="searchInput" />
                        </div>
                    </div>
                 </div>
                 <!-- Filters and Lesson Listings -->
                <div class="row">
                    <!-- Filters -->
                    <div class="col-lg-3">
                        <div class="row">
                            <div class="col-sm-6 col-lg-6 col-md-6 col-6">
                                  <select name="sort" @change="$emit('update:sortOrder',event.target.value)">
                                    <option>--Sort By--</option>
                                    <option value="title">Title</option>
                                    <option value="price">Price</option>
                                    <option value="location">Location</option>
                                    <option value="availableSpaces">Spaces</option>
                                </select>
                            </div>
                            <div class="col-sm-6 col-lg-6 col-md-6 col-6">
                          <div class="form-check">
                            <label><input type="radio" @change="$emit('update:sortOrder',event.target.value)" value="Ascending" id="ascending"> Ascending</label>
                          </div>
                          <div class="form-check">
                            <label><input type="radio" @change="$emit('update:sortOrder',event.target.value)" value="Descending" id="descending"> Descending</label>
                          </div>
                            </div>
                        </div>
                         
                    </div>
                    <!-- Lesson Listings -->
                    <div class="col-lg-9">
                        <div class="row wrapper">                   
                            <div class="col-lg-4 col-md-6 col-sm-6 col-12 box" v-for="lesson in filterSessions()">
                               <div class="card my-3 product">
                                  <div class="card-image">
                                   <div class="wrap">
                                     <img class="img-fluid" v-bind:src="lesson.image" alt="image 1">
                                  </div><!-- card image -->
                                  </div> 
                                  <div class="card-content bg-light text-dark">
                                  <span class="card-title">{{lesson.title}}</span>
                                  <hr>
                                
                                  <span>{{lesson.location}}</span>
                                  <span style="float: right;">{{lesson.price}}</span>
                                  <br>
                                  <br>
                                  <span class="pt-5">Available Spaces {{lesson.availableSpaces}}</span>
                                  <br>
                                  <br>
                                  <div class="row">
                                    <div class="col-9 col-lg-9 col-md-9 col-sm-9">
                                        <button type="button" id="show" data-target="show1" class="show btn btn-custom float-right" aria-label="Left Align" v-on:click="$emit('add-to-cart',lesson)" v-if="canAddToCart(lesson)"><i class="fa-solid fa-cart-plus"></i> Add to cart</button>
                                  <button disabled="disabled" v-else type="button" id="show" data-target="show1" class="show btn btn-custom float-right" aria-label="Left Align"><i class="fa-solid fa-cart-plus"></i>Empty</button>
                                    </div>
                                  </div>
                                  
                                  
                                 </div><!-- card content -->
                                 <div class="card-reveal show1">
                                 <span class="card-title">Card Title</span> <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">×</span></button>
                                  <p>Here is some more information about this product that is only revealed once clicked on.</p>
                               </div><!-- card reveal -->
                                </div>
                            </div>
                        </div>
                   
                    </div>
                </div>
</template>

<script>
export default {
  name: 'lesson-component',
  props:
    {
      lessons: {type: Array, required: true}
    }

    

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
