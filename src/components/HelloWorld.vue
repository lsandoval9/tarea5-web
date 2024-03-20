<template>
  <v-app>
    <v-container>
      <!-- Sección superior -->
      <v-text-field v-model="book.name" label="Nombre"></v-text-field>
      <v-text-field v-model="book.author" label="Autor"></v-text-field>
      <v-text-field v-model="book.year" label="Año"></v-text-field>
      <v-select v-model="book.genres" :items="genres" label="Géneros" multiple></v-select>
      <v-btn @click="addBook">Agregar</v-btn>

      <!-- Sección inferior -->
      <v-data-table :headers="headers" :items="books" class="elevation-1">
        <template v-slot:item.genres="{ item }">
          {{ item.genres.join(', ') }}
        </template>
        <template v-slot:item.actions="{ item }">
          <v-icon size="small" @click="deleteBook(item)">
            mdi-delete
          </v-icon>
        </template>
      </v-data-table>
    </v-container>
  </v-app>
</template>

<script>
import { ref, onMounted } from 'vue'
import 'firebase/firestore'

// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { getFirestore, collection, getDocs, addDoc } from 'firebase/firestore/lite';

// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
const firebaseConfig = {
  apiKey: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  authDomain: "xxxxxxxxxxxxxxxxxxxxxxxxxx",
  projectId: "xxxxxxxxxxxxxxxxxxxxxxxxxxx",
  storageBucket: "xxxxxxxxxxxxxxxxxxxxxxx",
  messagingSenderId: "xxxxxxxxxxxxxxxxxxx",
  appId: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

export default {
  setup() {
    const book = ref({ name: '', author: '', year: '', genre: '' })
    const genres = ref(['Ficción', 'No ficción', 'Ciencia ficción', 'Fantasía', 'Misterio', 'Biografía', 'Terror', 'Romance'])
    const books = ref([])
    const headers = ref([
      { title: 'Nombre', key: 'name' },
      { title: 'Autor', key: 'author' },
      { title: 'Año', key: 'year', sortable: true },
      { title: 'Género', key: 'genres' },
      { title: 'Acciones', key: 'actions' }
    ])

    const addBook = async () => {
      try {
        const docRef = await addDoc(collection(db, "books"), book.value);
        book.value.id = docRef.id;
        books.value.push(book.value);
        book.value = { name: '', author: '', year: '', genre: '' };
      } catch (e) {
        console.error("Error adding document: ", e);
      }
    }

    const deleteBook = (item) => {

    }

    onMounted(async () => {
      const booksCol = collection(db, 'books');
      const booksSnapshot = await getDocs(booksCol);
      const booksList = booksSnapshot.docs.map(doc => doc.data())

      booksList.forEach(book => {
        books.value.push(book)
      })
    })

    return { book, genres, books, headers, addBook, deleteBook }
  }
}
</script>
