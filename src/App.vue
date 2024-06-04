<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input" /><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="btn btn-save">Save</button>
    </form>
    <ul class="articles-list">
      <li v-for="article in articles" :key="article.id" class="article">
        <h3>{{ article.title }}</h3>
        <p>{{ article.content }}</p>
        <button @click="edit(article)" class="btn btn-edit">Edit</button>
        <button @click="remove(article.id)" class="btn btn-delete">Delete</button>
      </li>
    </ul>
    <button @click="load" class="btn btn-load">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    });
    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
        console.log('Articles loaded:', articles.value);
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        console.log('Article saved:', response.data);

        if (form.id) {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          articles.value.push(response.data);
        }

        resetForm();
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter((article) => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    function resetForm() {
      form.id = null;
      form.title = '';
      form.content = '';
    }

    onMounted(load);

    return { form, articles, save, edit, remove, load };
  }
}
</script>

<style>
.container {
  max-width: 500px;
  margin: 50px auto;
  padding: 20px;
  font-family: Arial, sans-serif;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.form {
  margin-bottom: 20px;
}

.input,
.textarea {
  width: calc(100% - 20px);
  padding: 10px;
  margin: 5px auto;
  display: block;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.btn {
  width: calc(100% - 20px);
  padding: 10px;
  margin: 10px auto;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
  display: block;
  text-align: center;
}

.btn:hover {
  background-color: #ddd;
}

.btn-save {
  background-color: #4caf50;
  color: white;
}

.btn-edit {
  background-color: #2196f3;
  color: white;
}

.btn-delete {
  background-color: #f44336;
  color: white;
}

.btn-load {
  background-color: #ff9800;
  color: white;
}

.articles-list {
  list-style: none;
  padding: 0;
}

.article {
  padding: 10px;
  margin: 10px auto;
  background-color: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
}

.article h3 {
  margin: 0 0 5px;
}

.article p {
  margin: 0;
}
</style>
