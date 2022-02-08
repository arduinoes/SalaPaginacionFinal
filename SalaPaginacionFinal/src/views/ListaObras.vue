<template>
  <Navbar />
  <div class="row justify-content-center">
    <BaseTarjeta
      class="col-4 m-2"
      v-for="obra in obras"
      :key="obra.id"
      :obra="obra"
    />
  </div>
  <div class="d-flex justify-content-center">
    <nav>
      <ul class="pagination">
        <li class="d-flex page-item me-5">
          <button
            v-show="this.page != 1"
            class="page-link"
            @click.prevent="botonAnterior"
          >
            Anterior
          </button>
        </li>
        <li class="d-flex page-item me-5">
          <button
            v-show="this.page != this.paginas"
            class="page-link"
            @click.prevent="botonSiguiente"
          >
            Siguente
          </button>
        </li>
      </ul>
    </nav>
  </div>
</template>

<script>
import Navbar from "../components/Navbar.vue";
import BaseTarjeta from "../components/BaseTarjeta.vue";
import {
  collection,
  getDocs,
  query,
  startAfter,
  orderBy,
  limit,
  endBefore,
  limitToLast,
} from "firebase/firestore";
import { db } from "../main";
export default {
  name: "ListaObras",
  components: {
    Navbar,
    BaseTarjeta
  },
  data() {
    return {
      total: "",
      limit: 3,
      paginas: "",
      page: 1,
      lastVisible: "",
      firstVisible: "",
      obras: [],
      obra: {
        title: "",
        author: "",
        date: "",
        photo: "",
      },
    };
  },
  methods: {
    async obtenerTotalDocumentos() {
      const totalDatos = query(collection(db, "obras"));
      const documentSnapshots = await getDocs(totalDatos);
      const total = documentSnapshots.docs.length;
      this.total = total;
      this.paginas = Math.ceil(this.total / this.limit);
    },
    async obtenerDatos() {
      const first = query(
        collection(db, "obras"),
        orderBy("author"),
        limit(this.limit)
      );
      this.obtenerTotalDocumentos();
      const documentSnapshots = await getDocs(first);
      const lastVisible =
        documentSnapshots.docs[documentSnapshots.docs.length - 1] || null;
      const firstVisible = documentSnapshots.docs[0] || null;
      this.firstVisible = firstVisible;
      this.lastVisible = lastVisible;

      const querySnapshot = await getDocs(first);
      querySnapshot.forEach((doc) => {
        let obra = doc.data();
        obra.id = doc.id;
        this.obras.push(obra);
        console.log(obra);
      });
    },
    async botonSiguiente() {
      const next = query(
        collection(db, "obras"),
        orderBy("author"),
        limit(this.limit),
        //offset(2),
        startAfter(this.lastVisible)
      );
      const documentSnapshots = await getDocs(next);
      const lastVisible =
        documentSnapshots.docs[documentSnapshots.docs.length - 1] || null;
      this.lastVisible = lastVisible;
      const firstVisible = documentSnapshots.docs[0] || null;
      this.firstVisible = firstVisible;
      this.page = this.page + 1;

      this.obras = [];
      const querySnapshot = await getDocs(next);
      querySnapshot.forEach((doc) => {
        let obra = doc.data();
        obra.id = doc.id;
        this.obras.push(obra);
        console.log("page", this.page);
      });
    },
    async botonAnterior() {
      const back = query(
        collection(db, "obras"),
        orderBy("author"),
        limitToLast(this.limit),
        endBefore(this.firstVisible)
      );

      const documentSnapshots = await getDocs(back);
      const lastVisible =
        documentSnapshots.docs[documentSnapshots.docs.length - 1] || null;
      this.lastVisible = lastVisible;
      const firstVisible = documentSnapshots.docs[0] || null;
      this.firstVisible = firstVisible;
      this.page = this.page - 1;

      this.obras = [];
      const querySnapshot = await getDocs(back);
      querySnapshot.forEach((doc) => {
        let obra = doc.data();
        obra.id = doc.id;
        this.obras.push(obra);
      });
    },
  },
  mounted() {
    this.obtenerDatos();
  },
};
</script>

<style>
.text-center {
  text-align: center;
}
</style>
