<template>
  <div class="container">
                    <div class="checkout">
                        <!-- Order Summary -->
                        <div class="row">
                            <div class="col-md-9">
                                <h2>Order Summary</h2>
                                <!-- Displaying items in the cart -->
                                <div class="row">
                                    <div class="col-md-4 col-sm-6 col-12" v-for="lesson in checkOut()">
                                        <div class="card my-3">
                                            <!-- Display image of the lesson -->
                                            <div class="card-image">
                                             <div class="wrap">
                                               <img class="img-fluid" v-bind:src="lesson.lessons.image" alt="image 1">
                                            </div><!-- card image -->
                                            </div> 
                                            <div class="card-content bg-light text-dark">
                                            <span class="card-title">{{lesson.lessons.title}}</span>         
                                            <hr>
                                            <span>{{lesson.lessons.location}}</span>
                                            <span style="float: right;">Price {{lesson.lessons.price}}</span>
                                            <br>
                                            <br>
                                            <!-- Button to remove item from the cart -->
                                            <div class="row">
                                              <div class="col-9 col-lg-9 col-md-9 col-sm-9">
                                                <button class="btn btn-danger"
                                                v-on:click="removeCartItem(lesson)"><small>Remove</small></button>
                                              </div>
                                            </div>
                                          </div>
                                    </div>  
                                </div>
                                <!-- Displayed when the cart is empty -->
                                <div v-if="checkOut() == ''">
                                    <div class="text-center mt-4">
                                        <img src="images/zero.png" alt="" width="160" height="160">
                                        <p class="mt-2"><strong>Cart Empty</strong></p>
                                    </div>
                                </div>
                            </div>
                            </div>
                            <!-- Billing Address Section -->
                            <div class="col-md-3">
                                <div class=" order-md-2 mb-4">
                                    <h4 class="d-flex justify-content-between align-items-center mb-3">
                                        <span class="text-muted">Billing Address</span>
                                        <span class="badge badge-secondary badge-pill">3</span>
                                    </h4>
                                        <ul class="list-group mb-3">
                                            <label for="firstName">Full name</label>
                                            <li class="list-group-item d-flex justify-content-between lh-condensed">
                                                
                                                <input type="text" class="form-control" id="firstName" placeholder="" v-model="name" v-on:blur="validateName" value="" required>
                                                <div class="invalid-feedback">
                                                    <span v-if="nameError">{{ nameError }}</span>
                                                </div>
                                            </li>
                                            <br>
                                            <label for="lastName">Phone Number</label>
                                            <li class="list-group-item d-flex justify-content-between lh-condensed">
                                            <input type="number" class="form-control" id="lastName" v-model="phone" v-on:blur="validatePhone" placeholder="" value="" required>
                                            </li>
                                        </ul>
                                        <hr class="mb-4">
                                        <button v-if="!isValid" class="btn btn-primary btn-lg btn-block"disabled type="submit">Continue to checkout</button>
                                        <button v-else class="btn btn-primary btn-lg btn-block" v-on:click="submitForm" type="submit">Continue to checkout</button>
                                </div>
                            </div>
                        </div>
                    <!-- Displaying items in the cart -->
                    <div class="row">
                        <div class="col-md-8 order-md-2 mb-4">
                            <h4 class="d-flex justify-content-between align-items-center mb-3">
                                <span class="text-muted">Your cart</span>
                                <span class="badge badge-secondary badge-pill">3</span>
                            </h4>
                            <ul class="list-group mb-3">
                                <!-- Displaying name and phone number -->
                                <li class="list-group-item d-flex justify-content-between lh-condensed">
                                    <div>
                                        <h6 class="my-0">Name</h6>

                                    </div>
                                    <span class="text-muted">{{name}}</span>
                                </li>
                                <li class="list-group-item d-flex justify-content-between lh-condensed">
                                    <div>
                                        <h6 class="my-0">Number</h6>

                                    </div>
                                    <span class="text-muted">{{phone}}</span>
                                </li>
                            </ul>
                        </div>
                       
                    </div>
                </div>
            </div>
</template>

<script>
export default {
  name: 'checkout-component',
  props: {
    
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
