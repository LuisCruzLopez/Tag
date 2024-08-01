<template>
  <div>
    <label class="typo__label">Para</label>
    <multiselect
      v-model="selectedPerson"
      placeholder="Escribe un nombre o puesto"
      label="displayLabel"
      track-by="id"
      :options="filteredOptions"
      :multiple="true"
      :taggable="false"
      :searchable="true"
      @search-change="onSearchChange"
      @select="onSelect"
      @remove="onRemove"
      :show-no-results="false"
    >
      <template #option="{ option }">
        <div class="custom-option">
          <img v-if="option.imagen" :src="option.imagen" alt="Imagen" class="option-image" />
          <div class="option-info">
            <span class="option-name">{{ option.displayLabel }}</span>
            <span v-if="option.detalle" class="option-detail">{{ option.detalle }}</span>
          </div>
        </div>
      </template>
      <!-- <template #tag="{ option, remove }">
        <span :class="['custom-tag', option.asignatura ? 'tutor-tag' : 'student-tag']">
          <img v-if="option.imagen" :src="option.imagen" alt="Imagen" class="tag-image" />
          <span class="tag-name">{{ option.displayLabel }}</span>
          <button @click="remove(option)" class="remove-button">✖</button>
        </span>
      </template> -->
    </multiselect>
    <pre class="language-json"><code>{{ idsJson }}</code></pre>
  </div>
</template>

<script>
import Multiselect from 'vue-multiselect'
import escuelaData from '@/data.json'

export default {
  components: {
    Multiselect
  },
  data() {
    return {
      selectedPerson: [],
      options: [],
      searchQuery: '',
      idsJson: []
    }
  },
  created() {
    // Transformar los datos para incluir etiquetas y detalles
    this.options = [
      ...escuelaData.alumnos.map((item) => ({
        ...item,
        displayLabel: `${item.nombre} - ${item.edad} años`,
        detalle: item.grado ? `${item.grado} grado` : `Asignatura: ${item.asignatura}`
      })),
      ...escuelaData.profesores.map((item) => ({
        ...item,
        displayLabel: `${item.nombre} - ${item.edad} años`,
        detalle: item.asignatura ? `Asignatura: ${item.asignatura}` : null
      })),
      ...escuelaData.claves // Añadir las claves directamente desde el JSON
    ]
  },
  methods: {
    onSearchChange(query) {
      this.searchQuery = query.toLowerCase()
    },
    onSelect(option) {
      if (option.isKey) {
        const value = prompt(`Ingrese un valor para ${option.displayLabel}`)
        if (value) {
          const newOption = {
            id: `${option.id}=${value}`,
            nombre: `${option.id}=${value}`,
            displayLabel: `${option.displayLabel}=${value}`
          }
          this.selectedPerson.push(newOption)
          this.updateIdsJson(newOption)
        }
        // Remover la selección actual
        const index = this.selectedPerson.findIndex((p) => p.id === option.id)
        if (index !== -1) {
          this.selectedPerson.splice(index, 1)
        }
      } else {
        this.updateIdsJson(option)
      }
    },
    onRemove(option) {
      this.idsJson = this.idsJson.filter((id) => id !== option.id)
    },
    updateIdsJson(option) {
      if (!this.idsJson.includes(option.id)) {
        this.idsJson.push(option.id)
      }
    }
  },
  computed: {
    filteredOptions() {
      if (!this.searchQuery) return []
      return this.options.filter((option) =>
        Object.entries(option).some(
          ([key, val]) =>
            key.toLowerCase().includes(this.searchQuery) ||
            String(val).toLowerCase().includes(this.searchQuery)
        )
      )
    }
  }
}
</script>

<style src="vue-multiselect/dist/vue-multiselect.css"></style>

<style scoped>
.custom-option {
  display: flex;
  align-items: center;
}
.option-image {
  width: 50px;
  height: 50px;
  margin-right: 10px;
}
.option-info {
  display: flex;
  flex-direction: column;
}
.option-name {
  font-weight: bold;
}
.option-detail {
  font-size: 0.9em;
  color: gray;
}

.custom-tag {
  display: flex;
  align-items: center;
  border-radius: 3px;
  padding: 5px 10px;
  margin: 5px;
  font-size: 14px;
  color: white;
}
.student-tag {
  background-color: #28a745;
}
.tutor-tag {
  background-color: #dc3545;
}
.tag-image {
  width: 20px;
  height: 20px;
  margin-right: 5px;
}
.tag-name {
  margin-right: 5px;
}
.remove-button {
  background: none;
  border: none;
  color: white;
  font-size: 1.2em;
  cursor: pointer;
}
</style>
