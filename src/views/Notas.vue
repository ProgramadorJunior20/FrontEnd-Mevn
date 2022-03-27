<template>
  <div class="container">
    <h1>Notas</h1>
    <b-alert
      :show="dismissCountDown"
      dismissible
      :variant="mensaje.color"
      @dismissed="dismissCountDown = 0"
      @dismiss-count-down="countDownChanged"
    >
      {{ mensaje.texto }}
    </b-alert>

    <form @submit.prevent="editarNota(notaEditar)" v-if="editar">
      <h3>Editar Nota</h3>
      <input
        type="text"
        class="form-control my-2"
        placeholder="Nombre"
        v-model="notaEditar.nombre"
      />
      <input
        type="text"
        class="form-control my-2"
        placeholder="Descripción"
        v-model="notaEditar.descripcion"
      />
      <b-button class="btn-warning my-2 mx-2" type="submit">Confirmar</b-button>
      <b-button class="my-2" type="submit" @click="editar = false"
        >Cancelar</b-button
      >
    </form>

    <form @submit.prevent="agregarNota()" v-if="!editar">
      <h3>Agregar Nueva Nota</h3>
      <input
        type="text"
        class="form-control my-2"
        placeholder="Nombre"
        v-model="nota.nombre"
      />
      <input
        type="text"
        class="form-control my-2"
        placeholder="Descripción"
        v-model="nota.descripcion"
      />
      <b-button class="btn-success my-2 btn-block" type="submit"
        >Agregar</b-button
      >
    </form>

    <table class="table table-striped">
      <thead>
        <tr>
          <th scope="col">id</th>
          <th scope="col">Nombre</th>
          <th scope="col">Descripción</th>
          <th scope="col">Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in notas" :key="index">
          <th scope="row">{{ item._id }}</th>
          <td>{{ item.nombre }}</td>
          <td>{{ item.descripcion }}</td>
          <td>
            <!-- <b-button @click="alerta()">Acción</b-button> -->
            <b-button
              @click="eliminarNota(item._id)"
              class="btn-danger btn-sm mx-2"
              >Eliminar</b-button
            >
            <b-button
              @click="activarEditar(item._id)"
              class="btn-warning btn-sm"
              >Editar</b-button
            >
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Paginacion -->
    <nav aria-label="Page navigation example">
      <ul class="pagination">
        <li class="page-item" :class="{ disabled: paginaActual === 1 }">
          <router-link
            class="page-link"
            :to="{ query: { pagina: paginaActual - 1 } }"
            >Anterior</router-link
          >
        </li>
        <li
          class="page-item"
          :class="{ active: paginaActual === index + 1 }"
          v-for="(item, index) in cantidadPaginas"
          :key="index"
        >
          <router-link
            :to="{ query: { pagina: index + 1 } }"
            class="page-link"
            >{{ index + 1 }}</router-link
          >
        </li>
        <li
          class="page-item"
          :class="{ disabled: paginaActual === cantidadPaginas }"
        >
          <router-link
            class="page-link"
            :to="{ query: { pagina: paginaActual + 1 } }"
            >Siguiente</router-link
          >
        </li>
      </ul>
    </nav>

    <p>
      Total notas: {{ totalNotas }} - Cantidad de páginas: {{ cantidadPaginas }}
    </p>
  </div>
</template>

<script>
import { mapState } from "vuex";
export default {
  data() {
    return {
      totalNotas: 0,
      limite: 5,
      paginaActual: 1,
      notas: [],
      mensaje: { color: "", texto: "" },
      dismissSecs: 5,
      dismissCountDown: 0,
      nota: { nombre: "", descripcion: "" },
      editar: false,
      notaEditar: {},
    };
  },
  watch: {
    "$route.query.pagina": {
      immediate: true,
      handler(pagina) {
        pagina = parseInt(pagina) || 1;
        /* console.log("pagina", pagina); */
        this.paginacion(pagina);
        this.paginaActual = pagina;
      },
    },
  },
  computed: {
    ...mapState(["token"]),
    cantidadPaginas() {
      return Math.ceil(this.totalNotas / this.limite);
    },
  },
  /* created(){
        this.listarNotas();

    }, */

  methods: {
    paginacion(pagina) {
      let config = {
        headers: {
          "access-token": this.token,
        },
      };

      let skip = (pagina - 1) * this.limite;

      this.axios
        .get(`/nota?limite=${this.limite}&skip=${skip}`, config)
        .then((res) => {
          this.notas = res.data.notaDB;
          this.listarNotas = res.data.totalNotas;
        })
        .catch((e) => {
          console.log(e.response);
        });
    },
    /* alerta(){
            this.mensaje.color = 'danger';
            this.mensaje.texto = 'Probando alerta';
            this.showAlert();
        }, */

    /* listarNotas(){
            let config = {
                headers: {
                    'access-token': this.token
                }
            }

            this.axios.get('/nota', config)
                .then(res => {
                    console.log(res.data.notaDB);
                    this.notas = res.data.notaDB;
                })
                .catch(e => {
                    console.log(e.response);
                })
        }, */
    agregarNota() {
      let config = {
        headers: {
          "access-token": this.token,
        },
      };

      /* console.log(this.nota); */
      this.axios
        .post("/nueva-nota", this.nota, config)
        .then((res) => {
          this.notas.push(res.data);
          this.nota.nombre = "";
          this.nota.descripcion = "";
          this.mensaje.color = "success";
          this.mensaje.texto = "Nota Agredada!";
          this.showAlert();
        })
        .catch((e) => {
          console.log(e.response);
          if (e.response.data.error.errors.nombre.message) {
            this.mensaje.texto = e.response.data.error.errors.nombre.message;
          } else {
            this.mensaje.texto = "Error de sistema";
          }
          this.mensaje.color = "danger";
          this.showAlert();
        });
    },
    eliminarNota(id) {
      /* console.log(id); */
      this.axios
        .delete(`/nota/${id}`)
        .then((res) => {
          const index = this.notas.findIndex(
            (item) => item._id === res.data._id
          );
          this.notas.splice(index, 1);
          this.mensaje.color = "success";
          this.mensaje.texto = "Nota Eliminada";
          this.showAlert();
        })
        .catch((e) => {
          console.log(e.response);
        });
    },
    activarEditar(id) {
      this.editar = true;
      /* console.log(id); */
      this.axios
        .get(`/nota/${id}`)
        .then((res) => {
          this.notaEditar = res.data;
        })
        .catch((e) => {
          console.log(e.response);
        });
    },
    editarNota(item) {
      this.axios
        .put(`/nota/${item._id}`, item)
        .then((res) => {
          const index = this.notas.findIndex((n) => n._id === res.data._id);
          this.notas[index].nombre = res.data.nombre;
          this.notas[index].descripcion = res.data.descripcion;
          this.mensaje.color = "success";
          this.mensaje.texto = "Nota Editada";
          this.showAlert();
          this.editar = false;
        })
        .catch((e) => {
          console.log(e.response);
        });
    },
    countDownChanged(dismissCountDown) {
      this.dismissCountDown = dismissCountDown;
    },
    showAlert() {
      this.dismissCountDown = this.dismissSecs;
    },
  },
};
</script>
