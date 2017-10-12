<template>
  <div class="hello">
    <h1 class="title has-text-left">&nbsp;Recipes</h1>
    <button class="button is-dark" v-on:click="toggleModal">Add one</button>
    <div class="section recipe-list">
      <div class="recipe" v-if="recipes" v-for="recipe in recipes">
        <div class="box has-text-left">
          <i class="is-pulled-left fa fa-fw fa-4x" v-bind:class="{'fa-star': isSelected[recipe._id], 'has-text-warning': isSelected[recipe._id], 'fa-star-o': !isSelected[recipe._id]}" v-on:click="toggleStar(recipe._id)"></i>
          <div class="recipe-title has-text-weight-bold is-size-2">{{recipe.name}}</div>
          <div class="recipe-ingredients is-size-4" v-if="recipe.ingredients">
            needs &ndash;&nbsp;<em>{{recipe.ingredients}}</em>
          </div>
        </div>
      </div>
    </div>
    <div v-if="recipes.length == 0">
      There are currently no recipes to view.<br /><br />
      <button class="button is-dark" v-on:click="toggleModal">Add one</button>
    </div>
    <!--modal-->
    <div class="modal" v-bind:class="{ 'is-active': modalActive }">
      <div class="modal-background"></div>
      <div class="modal-content">
        <section class="modal-card-body">    
          <h1 class="title is-pulled-left">Add Recipe</h1>
          <div class="field">
            <p class="control">
              <input class="input" type="text" name="recipe_title" placeholder="Recipe Title" v-model="recipeTitle" v-validate="'required'">
              <span v-show="errors.has('password')" class="help is-danger"><i class="fa fa-warning"></i>&nbsp;{{ errors.first('password') }}</span>
            </p>
          </div>
          <div class="field">
            <p class="control">
              <input class="input" type="text" name="recipe_ingredients" placeholder="Ingredients (non-pantry items)" v-model="recipeIngredients">
            </p>
          </div>   
          <div class="field">
            <p class="control is-pulled-right">
              <button class="button is-info" v-on:click="validateBeforeSubmit">
                Save
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
</template>

<script>
import axios from 'axios'

export default {
  name: 'hello',
  data () {
    return {
      recipes: [],
      modalActive: false,
      recipeTitle: '',
      recipeIngredients: '',
      selectedRecipes: [],
      isSelected: []
    }
  },
  created () {
    this.init();
  },
  methods:{
    init (){
      axios.get('http://localhost:3005/v1/recipe')
      .then(response => {
        console.log(response);
        this.recipes = response.data;
      })
      .catch(error => {
        console.log(error);
      })
    },
    toggleModal () {
      if(this.modalActive){
        this.modalActive = false;
        this.recipeTitle = '';
        this.recipeIngredients = '';
      }else{
        this.modalActive = true;
      }
    },
      validateBeforeSubmit() {
        this.$validator.validateAll().then((result) => {
          if (result) {
            this.addRecipe()
            return;
          }
        });
      },
      addRecipe () {
        axios.post('http://localhost:3005/v1/recipe/add',{
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
        })
      },
      toggleStar (id) {
        console.log(this.selectedRecipes.indexOf(id));
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
</style>