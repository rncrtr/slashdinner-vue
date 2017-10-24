<template>
  <div class="hello">
    <h1 class="title has-text-left no-print">&nbsp;Recipes</h1>
    <h1 class="title has-text-left no-screen">&nbsp;Meals List &ndash; {{niceDate}}</h1>
    <div class="section" style="padding-top: 0px;">
      <button class="button is-link is-pulled-left no-print" v-show="showMealsList" v-on:click="toggleListView">Show All Recipes</button>
      <button class="button is-link is-pulled-left no-print" v-show="!showMealsList" v-on:click="toggleListView">Show Meals List</button>
      <button class="button is-dark is-pulled-right no-print" v-show="!showMealsList" v-on:click="toggleModal(null)">Add One</button>
      <button class="button is-info is-pulled-right no-print" v-show="showMealsList" v-on:click="printPage">Print List</button>
      <button class="button is-link is-pulled-right no-print" v-show="!showMealsList" v-on:click="clearSelected">Clear Selected</button>
      <br /><br />
      <div class="recipe-list">
        <div class="recipe" v-if="recipes" v-for="recipe in filteredRecipes" v-bind:class="{'separator': showMealsList}">
          <div class="has-text-left" v-bind:class="{'box': !showMealsList }">
            <i class="is-pulled-left fa fa-fw fa-4x" v-show="!showMealsList" v-bind:class="{'fa-star': isSelected[recipe._id], 'has-text-warning': isSelected[recipe._id], 'fa-star-o': !isSelected[recipe._id]}" v-on:click="toggleStar(recipe._id)"></i>
            <i class="fa fa-fw fa-times fa-3x has-text-danger is-pulled-right" v-show="!showMealsList" v-on:click="deleteRecipe(recipe._id)"></i>
            <i class="fa fa-fw fa-pencil fa-3x has-text-info is-pulled-right" v-show="!showMealsList" v-on:click="toggleModal(recipe)"></i>
            
            <div class="recipe-title has-text-weight-bold" v-bind:class="{'is-size-2': !showMealsList }">{{recipe.name}}</div>
            <div class="recipe-ingredients" v-bind:class="{'is-size-4': !showMealsList }" v-if="recipe.ingredients">
              &ndash;&nbsp;<em>{{recipe.ingredients}}</em>
            </div>
          </div>
        </div>
      </div>
      <div v-if="recipes.length == 0">
        There are currently no recipes to view.<br /><br />
        <button class="button is-dark" v-on:click="toggleModal(null)">Add one</button>
      </div>
      <!--modal-->
      <div class="modal" v-bind:class="{ 'is-active': modalActive }">
        <div class="modal-background"></div>
        <div class="modal-content">
          <section class="modal-card-body">    
            <h1 class="title is-pulled-left" v-if="!isEdit">Add Recipe</h1>
            <h1 class="title is-pulled-left" v-if="isEdit">Edit Recipe</h1>
            <div class="field">
              <p class="control">
                <input class="input" type="text" name="recipe_title" placeholder="Recipe Title" v-model="recipeTitle" v-validate="'required'">
                <span v-show="errors.has('recipeTitle')" class="help is-danger"><i class="fa fa-warning"></i>&nbsp;{{ errors.first('recipeTitle') }}</span>
              </p>
            </div>
            <div class="field">
              <p class="control">
                <input class="input" type="text" name="recipe_ingredients" placeholder="Ingredients (non-pantry items)" v-model="recipeIngredients">
              </p>
            </div>   
            <div class="field">
              <p class="control is-pulled-right">
                <button class="button is-info" v-on:click="validateBeforeSubmit(null)" v-if="!isEdit">
                  Save
                </button>
                <button class="button is-info" v-on:click="validateBeforeSubmit(recipeId)" v-if="isEdit">
                  Update
                </button>
                <button class="button is-default" v-on:click="toggleModal">Cancel</button>
              </p>
            </div>
          </section>
        </div>
        <button class="modal-close is-large" aria-label="close" v-on:click="toggleModal"></button>
      </div>
      <!--/modal-->
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import axios from 'axios'
import VueLocalStorage from 'vue-localstorage'
import moment from 'moment'
Vue.use(VueLocalStorage,{name: 'ls'})

export default {
  name: 'hello',
  data () {
    return {
      isLoggedIn: false,
      recipes: [],
      modalActive: false,
      isEdit: false,
      showMealsList: false,
      recipeId: '',
      recipeTitle: '',
      recipeIngredients: '',
      selectedRecipes: [],
      isSelected: [],
      moment: moment
    }
  },
  created () {
    //console.log(this.$ls.get('token'));
    if(!this.$ls.get('token')){
      this.$router.push('/login')
    }else{
      this.isLoggedIn = true
    }
    this.init();
    //console.log('gunna get ls ',this.$ls.get('selectedRecipes'));
    if(this.$ls.get('selectedRecipes')){
      this.selectedRecipes = this.$ls.get('selectedRecipes').split(',')
      //console.log(this.selectedRecipes);
      for (var i = this.selectedRecipes.length - 1; i >= 0; i--) {
        this.isSelected[this.selectedRecipes[i]] = true
      }
      //console.log( this.isSelected );
    }
  },
  computed:{
    filteredRecipes : function(){
      if(this.showMealsList){
        return this.recipes.filter((recipe) =>{
          if(this.selectedRecipes.indexOf(recipe._id) !=-1){
            return true;
          }
        })
      }else{
        return this.recipes
      }
    },
    niceDate: function(){
      return moment(Date.now()).format('MM/DD/YYYY [at] h:mm a')
    }
  },
  methods:{
    init (){
      axios.get('/v1/recipe')
      .then(response => {
        //console.log(response);
        this.recipes = response.data;
      })
      .catch(error => {
        console.log(error);
      })
      //this.$ls.set('selectedRecipes','')
    },
    toggleModal (editItem) {
      if(this.modalActive){
        this.modalActive = false;
        this.recipeTitle = '';
        this.recipeIngredients = '';
      }else{
        if(editItem){
          this.isEdit = true
          this.recipeId = editItem._id
          this.recipeTitle = editItem.name;
          this.recipeIngredients = editItem.ingredients;
        }else{
          this.isEdit = false
        }
        this.modalActive = true;
      }
    },
      validateBeforeSubmit(editId) {
        this.$validator.validateAll().then((result) => {
          if (result) {
            this.addRecipe(editId)
            return;
          }
        });
      },
      addRecipe (editId) {
        if(editId){
          axios.put('/v1/recipe/'+editId,{
            title: this.recipeTitle,
            ingredients: this.recipeIngredients
          })
          .then(response => {
            console.log('recipe updated');
            this.recipeTitle = '';
            this.recipeIngredients = '';
            this.init()
            this.toggleModal()
          })
          .catch(error => {
            console.log(error);
          });
        }else{
          axios.post('/v1/recipe/add',{
            title: this.recipeTitle,
            ingredients: this.recipeIngredients
          })
          .then(response => {
            console.log('recipe added');
            this.recipeTitle = '';
            this.recipeIngredients = '';
            this.init()
            this.toggleModal()
          })
          .catch(error => {
            console.log(error);
          });
        }
      },
      deleteRecipe(deleteId) {
        var result = confirm("Are you sure you want to delete this recipe?")
        if(result){
          axios.delete('/v1/recipe/'+deleteId)
          .then(response => {
            console.log('recipe updated');
            this.recipeTitle = '';
            this.recipeIngredients = '';
            this.init()
          })
          .catch(error => {
            console.log(error);
          });
        }
      },
      toggleStar (id) {
        //console.log(this.selectedRecipes.indexOf(id));
        if(this.selectedRecipes.indexOf(id)!=-1){
          this.isSelected[id] = false;
          var ind = this.selectedRecipes.indexOf(id);
          this.selectedRecipes.splice(ind,1);
          var selind = this.isSelected.indexOf(id);
          this.isSelected.splice(selind,1);
        }else{
          var ind = null;
          this.isSelected.push(id);
          this.isSelected[id] = true;
          this.selectedRecipes.push(id);
        }
        if(this.selectedRecipes.length > 0){
          //console.log('setting ',this.selectedRecipes)
          this.$ls.set('selectedRecipes',this.selectedRecipes)
        }else{
          this.$ls.set('selectedRecipes','')
        }
      },
      toggleListView () {
        if(this.showMealsList){
          this.showMealsList = false
        }else{
          this.showMealsList = true
        }
      },
      clearSelected (){
        var result = confirm("Are you sure you want to wipe out the whole shopping list?")
        if(result){
          this.selectedRecipes = []
          this.$ls.set('selectedRecipes','')
          this.isSelected = []
        }
          
      },
      printPage (){
        window.print()
      }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
.separator{
  border-bottom: 1px solid #696969;
  padding-bottom: 10px;
  margin-top: 10px;
}
.separator:first-child{
  margin-top: 0px;
}
</style>