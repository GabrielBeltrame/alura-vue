<template>
  <div>
    <h1 class="centralizado">Cadastro</h1>
    <h2 class="centralizado" />

    <p v-show="message" class="centralizado">{{ message }}</p>

    <h2 v-if="foto._id" class="centralizado">Alterando</h2>
    <h2 v-else class="centralizado">Incluindo</h2>

    <form @submit.prevent="grava()">
      <div class="controle">
        <label for="titulo">TÍTULO</label>
        <validation-provider rules="required" v-slot="{ errors }">
          <input
            name="titulo"
            id="titulo"
            autocomplete="off"
            v-model="foto.titulo"
          />
          <span class="erro">{{ errors[0] }}</span>
        </validation-provider>
      </div>

      <div class="controle">
        <label for="url">URL</label>
        <input id="url" autocomplete="off" v-model.lazy="foto.url" />
        <imagem-responsiva
          v-show="foto.url"
          :url="foto.url"
          :titulo="foto.titulo"
        />
      </div>

      <div class="controle">
        <label for="descricao">DESCRIÇÃO</label>
        <textarea id="descricao" autocomplete="off" v-model="foto.descricao" />
      </div>

      <div class="centralizado">
        <meu-botao rotulo="GRAVAR" tipo="submit" />
        <router-link :to="{ name: 'home' }"
          ><meu-botao rotulo="VOLTAR" tipo="button"
        /></router-link>
      </div>
    </form>
  </div>
</template>

<script>
import ImagemResponsiva from "../shared/imagem-responsiva/ImagemResponsiva.vue";
import Botao from "../shared/botao/Botao.vue";
import Foto from "../../domain/foto/Foto";
import FotoService from "../../domain/foto/FotoService";
import { ValidationProvider, extend } from "vee-validate";
import { required } from "vee-validate/dist/rules";

extend("required", {
  ...required,
  message: "This field is required"
});

export default {
  components: {
    "imagem-responsiva": ImagemResponsiva,
    "meu-botao": Botao,
    ValidationProvider
  },

  data() {
    return {
      foto: new Foto(),
      id: this.$route.params.id,
      message: ""
    };
  },

  methods: {
    grava() {
      this.$validator.validateAll().then(success => {
        if (success) {
          this.service.cadastra(this.foto).then(
            () => {
              if (this.id) this.$route.push({ name: "home" });
              this.foto = new Foto();
            },
            err => (this.message = err.message)
          );
          this.foto = new Foto();
        }
      });
    }
  },

  created() {
    this.service = new FotoService(this.$resource);

    if (this.id) {
      this.service.busca(this.id).then(foto => (this.foto = foto));
    }
  }
};
</script>

<style scoped>
.centralizado {
  text-align: center;
}
.controle {
  font-size: 1.2em;
  margin-bottom: 20px;
}
.controle label {
  display: block;
  font-weight: bold;
}

.controle label + input,
.controle textarea {
  width: 100%;
  font-size: inherit;
  border-radius: 5px;
}

.centralizado {
  text-align: center;
}

.erro {
  color: red;
}
</style>
