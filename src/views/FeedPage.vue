<template>
  <div>
    <!-- Recipe Feed Content -->
    <v-container>
      <v-row>
        <h1 v-if="!recipes.length">There's no recipes</h1>
        <v-col cols="12" md="4" v-for="recipe in recipes" :key="recipe.id">
          <v-card>
            <v-img
              :src="recipe.image || recipePlaceholder"
              height="200px"
              cover
            ></v-img>
            <v-card-title>{{ recipe.title }}</v-card-title>
            <v-card-subtitle
              >Created at: {{ recipe.createdAt }}</v-card-subtitle
            >
            <v-card-subtitle
              >Author: {{ recipe?.createdBy?.username || 'Anonymous' }}</v-card-subtitle
            >
            <v-card-text>{{ recipe.description }}</v-card-text>
            <v-card-actions>
              <v-btn @click="handleDeleteRecipe(recipe.id)">Delete</v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import recipePlaceholder from "@/assets/recipe-placeholder.jpeg";
import moment from "moment";

export default {
  setup() {
    const recipes = ref([]);

    const fetchRecipes = async () => {
      const token = localStorage.getItem("token");
      const response = await fetch("http://localhost:3000/recipes", {
        headers: { Authorization: token },
      });
      const data = await response.json();
      recipes.value = data.map((recipe) => ({
        ...recipe,
        createdAt: moment(recipe.createdAt).format("MMMM DD, YYYY h:mm"),
      }));
    };

    const handleDeleteRecipe = async (recipeId) => {
      const token = localStorage.getItem("token");
      const response = await fetch(
        `http://localhost:3000/recipes/${recipeId}`,
        {
          method: "DELETE",
          headers: {
            Authorization: token,
            "Content-Type": "application/json",
          },
        }
      );

      if (response.ok) {
        fetchRecipes();
      } else {
        const errorData = await response.json();
        console.error(`Error deleting recipe: ${errorData.message}`);
      }
    };

    onMounted(() => {
      fetchRecipes();
    });

    return { recipes, recipePlaceholder, handleDeleteRecipe };
  },
};
</script>
