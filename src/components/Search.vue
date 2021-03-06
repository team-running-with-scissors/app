<template>
<div>
  <div id="container-whole" class="container-main">
    <div>
      <h2 class="list-header">Recipe Search</h2>
      <h4>It's simple to get started. Just choose a month, category of food, click on a food and see your results!</h4>
      <span class="food-type-button-container">
        <button :class="ingredientType === 0 ? active : '' " class="filter-button"
         @click="ingredientType = 0"  
         >All</button>
        <button :class="ingredientType === 1 ? active : '' " class="filter-button" 
        @click="ingredientType = 1"
        >Veggies</button>
        <button :class="ingredientType === 2 ? active : '' " class="filter-button" 
        @click="ingredientType = 2"
        >Fruit</button>
        <button :class="ingredientType === 3 ? active : '' " class="filter-button" 
        @click="ingredientType = 3"
        >Meat</button>
        <button :class="ingredientType === 4 ? active : '' " class="filter-button" 
        @click="ingredientType = 4"
        >Seafood</button>
      </span><br/>
      <div id="month-holder" v-if="months">
        <button
        v-for="month in months"
        :key="month.id"
        @click="monthChoice = month.id; seasonColors()"
        :class="monthChoice === month.id ? activeSeason: ''"
        class="filter-button"
        >{{ month.month }}</button>
      </div>
      <a
        class="filter-button results"
        v-for="item in filteredIngredients"
        :key="item.index"
        @click="handleSearch(item.food)"
        :class="[searchTerm === item.food ? 'activeIngredient' : '' ]"
      >
        {{ item.food }}
      </a>
    </div>
    <div id="container-main">
      <ul v-if="searched">
        <li
          v-for="result in searchResults"
          :key="result.RecipeID"
          @click.prevent="handleView(result.RecipeID)"
        >
          <div id="image">
            <img :src="result.PhotoUrl">
          </div>
          <h1>{{ result.Title }}</h1>
          <p>{{ result.Description }}</p>
        </li>
      </ul>

      <recipe-card
        :selectedRecipe="selectedRecipe"
        :toggleRecipe="toggleRecipe"
        :addToMasterList="addToMasterList"
        :toggleZoom="toggleZoom"
        :userid="userid"
        :userShoppingList="userShoppingList"
        :removeFromMasterFavoriteList="removeFromMasterFavoriteList"
        
        v-if="recipeZoom"
        :addToMasterFavoriteList="addToMasterFavoriteList"
      />
    </div>
  </div>
</div>
</template>

<script>

import { searchRecipes, getRecipe, getFoods, getMonths } from '../../services/api.js';
import RecipeCard from './RecipeCard.vue';

export default {
  props: {
    addToMasterList: {
      type: Function,
      required: true
    },
    addToMasterFavoriteList: {
      type: Function,
      required: true
    },
    removeFromMasterFavoriteList: {
      type: Function,
      required: true
    },
    toggleZoom: {
      type: Function,
      required: true
    },
    userid: Number,
    userShoppingList: Array
  },
  components: {
    RecipeCard
  },
  data() {
    return {
      searchTerm: '',
      searchResults: null,
      searched: false,
      seasonalIngredients: [],
      recipeZoom: false,
      selectedRecipe: null,
      ingredientType: 0,
      active: 'active',
      monthChoice: null,
      months: null,
      activeSeason: null,
      highlight: 'highlight'
    };
  },
  created() {
    getFoods()
      .then(ingredient => {
        this.seasonalIngredients = ingredient;
      });
    getMonths()
      .then(month => {
        this.months = month;
      });
    const d = new Date;
    const n = d.getMonth();
    this.monthChoice = n + 1;
    this.activeSeason = this.seasonColors();
  },

  methods: {
    handleSearch(search) {
      this.searchTerm = search;
      console.log('in the search diddle');
      searchRecipes(search)
        .then(result => {
          this.searchResults = result.Results;
          this.searched = true;
        });
    },
    handleView(recipe) {
      console.log(recipe);
      // Make another API call, then send the returned data to the RecipeCard component/page
      getRecipe(recipe)
        .then(result => {
          this.recipeZoom = true;
          this.selectedRecipe = result;
        });
    },
    toggleRecipe(newUser) {
      if(newUser) {
        this.toggleZoom();
      }
      else {
        this.recipeZoom = !this.recipeZoom;
      }
    },
    seasonColors() {
      if(this.monthChoice === 12 || this.monthChoice < 3) {
        return this.activeSeason = 'winter';
      }
      else if(this.monthChoice > 2 && this.monthChoice < 6) {
        return this.activeSeason = 'spring';
      }
      else if(this.monthChoice > 5 && this.monthChoice < 9) {
        return this.activeSeason = 'summer';
      }
      else if(this.monthChoice > 8 && this.monthChoice < 12) { 
        return this.activeSeason = 'autum';
      }
    }
  },

  computed: {

    monthFilteredIngredients() {
      return this.seasonalIngredients.filter(ingredient => {
        return (this.monthChoice === 0 || this.monthChoice === ingredient.month_id);
      });
    },

    filteredIngredients() {
      return this.monthFilteredIngredients.filter(ingredient => {
        return (this.ingredientType === 0 || this.ingredientType === ingredient.type_id);
      });
    },
  }
};
</script>

<style scoped>
.list-header {
  font-family: 'Sedgwick Ave', cursive;
  font-size: 2em;
  color: #fff;
  text-align: center;
}
#container-whole {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.search-link:hover {
  cursor: pointer;
  color: rgb(255, 201, 60);
}

ul {
  list-style: none;
}
h1 {
  float: right;
}
li {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  position: relative;
  border: 1px solid black;
  border-radius: 6px;
  padding: 6px;
  cursor: pointer;
  align-items: center;
  height: fit-content;
  margin-bottom: 5px;
}
img {
  vertical-align: middle;
  max-height: 100%;
}
#image {
  overflow: hidden;
  height: 100px;
  width: 100px;
}
.button-container {
  border: 2px solid darkgray;
  border-radius: 3px;
}
.filter-button {
  background-color: lightgray;
  opacity: .75;
  border: none;
  margin-bottom: 15px;
  padding: 5px;
}
.results {
  color: white;
  font-size: 1em;
  margin-left: 1px;
  margin-right: 1px;
  border-radius: 2px;
  background-color: rgba(222, 184, 135, 0);
}
.activeIngredient {
  opacity: 1;
  font-size: 1em;
  border: 1px dashed white;
  padding: 0px;
  padding-left: 7px;

}
.filter-button:hover {
  opacity: 1;
  cursor: pointer;
}
.highlight {
  background-color: rgb(255, 201, 60);
}
.active {
  background-color: rgb(90, 71, 56);
  color: white;
}
.winter {
  background-color: rgb(21, 82, 99);
  color: white;
}
.spring {
  background-color: rgb(3, 148, 60);
  color: white;
}
.summer {
  background-color: rgb(255, 201, 60);
  color: white;
}
.autum {
  background-color: rgb(255, 154, 59);
  color: white;
}
#search-box{
  margin-top: 25px;
}

</style>
