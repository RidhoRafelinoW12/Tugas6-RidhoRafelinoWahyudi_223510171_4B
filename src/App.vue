<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <h2>{{ form.id ? 'Edit Article' : 'New Article' }}</h2>
      <input type="text" v-model="form.title" placeholder="Title" class="input" /><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h3>{{ article.title }}</h3>
        <p>{{ article.content }}</p>
        <button @click="edit(article)" class="button">Edit</button>
        <button @click="deleteArticle(article)" class="button">Delete</button>
      </li>
    </ul>
    <button @click="load" class="button">Load Articles</button>
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
      content: '',
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
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

        if (form.id) {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          articles.value.push(response.data);
        }

        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    async function deleteArticle(article) {
      try {
        await axios.delete(`http://localhost:3000/articles/${article.id}`);
        articles.value = articles.value.filter((a) => a.id !== article.id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    onMounted(load);

    return { form, articles, save, edit, deleteArticle, load };
  },
};
</script>

<style scoped>
.container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f9f9f9;
    border-radius: 10px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    color: black;
    background-image: url('./pepe1.jpg'); /* tambahkan properti ini untuk background image */
    background-size: cover;
    background-position: center;
}

.form {
  margin-bottom: 20px;
}

.input, .textarea {
  width: 100%;
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.button {
  background-color: #007bff;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-top: 10px;
}

.button:hover {
  background-color: #0056b3;
}

.article-list {
  list-style: none;
  padding: 0;
}

.article-item {
  background-color: #fff;
  padding: 20px;
  margin-bottom: 10px;
  border-radius: 5px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
}

.article-item h3 {
  margin: 0;
  font-size: 1.2em;
  color: black;
}

.article-item p {
  margin: 10px 0 0;
  color: black;
}

.article-item .button {
  margin-top: 10px;
}
</style>