<template>
    <v-app>
      <v-container>
        <!-- Sección superior: Formulario para agregar o actualizar libro -->
        <v-text-field v-model="book.name" label="Nombre"></v-text-field>
        <v-text-field v-model="book.author" label="Autor"></v-text-field>
        <v-text-field v-model="book.year" label="Año"></v-text-field>
        <v-select v-model="book.genres" :items="genres" label="Géneros" multiple></v-select>
        <v-btn @click="saveBook">{{ editingBookId ? 'Actualizar' : 'Agregar' }}</v-btn>
  
        <!-- Sección inferior: Lista de libros -->
        <v-data-table :headers="headers" :items="books" class="elevation-1">
          <template v-slot:item.name="{ item }">
            {{ item.data?.name }}
          </template>
          <template v-slot:item.author="{ item }">
            {{ item.data?.author }}
          </template>
          <template v-slot:item.year="{ item }">
            {{ item.data?.year }}
          </template>
          <template v-slot:item.genres="{ item }">
            {{ item.data?.genres.join(', ') }}
          </template>
          <template v-slot:item.actions="{ item }">
            <!-- Al hacer clic en el icono de actualización, se carga el libro en el formulario -->
            <v-icon size="small" @click="editBook(item)">
              mdi-pencil
            </v-icon>
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
  import { initializeApp } from "firebase/app";
  import { 
    getFirestore, 
    collection, 
    getDocs, 
    addDoc, 
    deleteDoc, 
    updateDoc, 
    doc 
  } from 'firebase/firestore/lite';
  
  // Configuración de Firebase
  const firebaseConfig = {
    apiKey: "AIzaSyDlZYWdPtihliV6Cxw_MUCNwPVlmD3kgyQ",
    authDomain: "desarrollo-web-facyt.firebaseapp.com",
    projectId: "desarrollo-web-facyt",
    storageBucket: "desarrollo-web-facyt.firebasestorage.app",
    messagingSenderId: "878698040565",
    appId: "1:878698040565:web:8139f3cdf266d6fc3889bc"
  };
  
  const app = initializeApp(firebaseConfig);
  const db = getFirestore(app);
  
  export default {
    setup() {
      // Formulario para agregar o editar un libro
      const book = ref({ name: '', author: '', year: '', genres: [] });
      // Variable que indica si se está editando (almacena el id del libro a actualizar)
      const editingBookId = ref(null);
      const genres = ref([
        'Ficción', 'No ficción', 'Ciencia ficción', 
        'Fantasía', 'Misterio', 'Biografía', 'Terror', 'Romance'
      ]);
      const books = ref([]);
      const headers = ref([
        { title: 'Nombre', key: 'name' },
        { title: 'Autor', key: 'author' },
        { title: 'Año', key: 'year', sortable: true },
        { title: 'Género', key: 'genres' },
        { title: 'Acciones', key: 'actions' }
      ]);
  
      // Función que guarda un libro: agrega o actualiza según el modo en que se encuentre
      const saveBook = async () => {
        if (editingBookId.value) {
          // Actualización de un libro existente
          await updateDoc(doc(db, "books", editingBookId.value), book.value);
          // Actualizar el libro en la lista local
          const index = books.value.findIndex(b => b.id === editingBookId.value);
          if (index !== -1) {
            books.value[index].data = { ...book.value };
          }
          editingBookId.value = null;
        } else {
          // Agregar un nuevo libro
          const docRef = await addDoc(collection(db, "books"), book.value);
          books.value.push({ id: docRef.id, data: { ...book.value } });
        }
        // Limpiar el formulario
        book.value = { name: '', author: '', year: '', genres: [] };
      };
  
      // Función para cargar los datos de un libro en el formulario y activar el modo edición
      const editBook = (item) => {
        book.value = { ...item.data };
        editingBookId.value = item.id;
      };
  
      // Función para eliminar un libro
      const deleteBook = async (item) => {
        await deleteDoc(doc(db, "books", item.id));
        books.value = books.value.filter(b => b.id !== item.id);
      };
  
      onMounted(async () => {
        const booksCol = collection(db, 'books');
        const booksSnapshot = await getDocs(booksCol);
        const booksList = booksSnapshot.docs.map(doc => ({
          id: doc.id, 
          data: doc.data()
        }));
  
        books.value = booksList;
      });
  
      return { book, genres, books, headers, saveBook, deleteBook, editBook, editingBookId }
    }
  }
  </script>
  