<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated class="bg-red text-white" style="background-color: #ff0000;">
      <q-toolbar>
        <q-btn flat round icon="menu" @click="toggleLeftDrawer" />
        <q-toolbar-title>
          ARTIKEL
        </q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="container q-pa-md">
        <q-card class="q-pa-md q-mb-md">
          <q-card-section>
            <q-form @submit.prevent="save">
              <q-input
                v-model="form.title"
                label="Title"
                outlined
                dense
                class="q-mb-md"
                :rules="[val => !!val || 'Title is required']"
              />
              <q-input
                v-model="form.content"
                label="Content"
                type="textarea"
                outlined
                dense
                class="q-mb-md"
                :rules="[val => !!val || 'Content is required']"
              />
              <q-btn type="submit" label="Save" color="primary" class="q-mb-md" />
            </q-form>
          </q-card-section>
        </q-card>

        <q-card class="q-pa-md q-mb-md">
          <q-card-section>
            <q-list bordered padding>
              <q-item
                v-for="article in articles"
                :key="article.id"
                clickable
                class="q-mb-xs"
              >
                <q-item-section>
                  <q-item-label>{{ article.title }}</q-item-label>
                  <q-item-label caption>{{ article.content }}</q-item-label>
                </q-item-section>
                <q-item-section side>
                  <q-btn dense icon="edit" @click.stop="edit(article)" />
                  <q-btn dense icon="delete" color="negative" @click.stop="remove(article.id)" />
                </q-item-section>
              </q-item>
            </q-list>
          </q-card-section>
        </q-card>

        <q-btn label="Load Articles" color="primary" @click="load" />

        <footer class="footer q-pa-md q-mt-xl text-white text-center">
          <q-separator class="q-mt-md q-mb-md" />
          <p>&copy; 2024 Integrasi Dengan API. All rights reserved.</p>
          <p>Contact Person: muhammaddhandhyputerairvantie@student.uir.ac.id</p>
        </footer>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue';
import axios from 'axios';
import { QLayout, QHeader, QToolbar, QToolbarTitle, QPageContainer, QPage, QForm, QInput, QBtn, QList, QItem, QItemSection, QItemLabel, QCard, QCardSection, QSeparator } from 'quasar';

// JSONBin API details
const JSONBIN_API_KEY = '$2a$10$XVTGLAfQa.7V85WP/L9GtObAB8BdXzLtaGmmqvq7LvynjKwzvLuPe';
const BIN_ID = '6659f4f5acd3cb34a850b41a';

const form = reactive({
  id: null,
  title: '',
  content: '',
});

const articles = ref([]);

async function load() {
  try {
    const response = await axios.get(`https://api.jsonbin.io/v3/b/${BIN_ID}/latest`, {
      headers: {
        'X-Master-Key': JSONBIN_API_KEY,
      },
    });
    articles.value = response.data.record.articles || [];
  } catch (error) {
    console.error('Error loading articles', error);
  }
}

async function save() {
  try {
    let updatedArticles;
    if (form.id) {
      // Update the existing article
      updatedArticles = articles.value.map((article) =>
        article.id === form.id ? { ...form } : article
      );
    } else {
      // Create a new article
      const newArticle = { ...form, id: Date.now() };
      updatedArticles = [...articles.value, newArticle];
    }
    await axios.put(
      `https://api.jsonbin.io/v3/b/${BIN_ID}`,
      { articles: updatedArticles },
      {
        headers: {
          'Content-Type': 'application/json',
          'X-Master-Key': JSONBIN_API_KEY,
        },
      }
    );
    form.id = null;
    form.title = '';
    form.content = '';
    await load();
  } catch (error) {
    console.error('Error saving article', error);
  }
}

function edit(article) {
  form.id = article.id;
  form.title = article.title;
  form.content = article.content;
}

async function remove(id) {
  try {
    const updatedArticles = articles.value.filter((article) => article.id !== id);
    await axios.put(
      `https://api.jsonbin.io/v3/b/${BIN_ID}`,
      { articles: updatedArticles },
      {
        headers: {
          'Content-Type': 'application/json',
          'X-Master-Key': JSONBIN_API_KEY,
        },
      }
    );
    await load();
  } catch (error) {
    console.error('Error deleting article', error);
  }
}

function toggleLeftDrawer() {
  
}

onMounted(load);
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: auto;
}

footer {
  background-color: #ff0000; 
  color: white; 
  width: 100%; 
  font-size: medium;
}
.q-footer p {
  margin: 6px 0; 
}

.q-footer .contact-info {
  font-size: 12px;
}

.q-footer .contact-info strong {
  font-weight: bolder;
}
</style>  
