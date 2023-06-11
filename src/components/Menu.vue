<template>
    <v-btn 
        @click="isActive = !isActive" 
        class="menu-button ma-2" 
        icon="mdi-menu">
    </v-btn>
    <div v-if="isActive" class="sidebar">
      <v-list>
        <v-list-item v-for="(value, key) in data" :id="key" >
        {{ value }}
        </v-list-item>
      </v-list>
        HELLO!
    </div>
</template>
  
<script>
export default {
  props: ['data'],
  data() {
    return {
      isActive: false
    };
  },
  methods: {
    toggleSidebar() {
      this.isActive = !this.isActive;
    },
  },
};
</script>

<style scoped>
.menu-button {
    z-index: 999;
    position: fixed;
    top: 0;
    opacity: 0.5;
}

.sidebar {
  position: fixed;
  top: 0;
  left: 0px;
  width: 200px;
  height: 100vh;
  background-color: #f1f1f1;
  z-index: 1000;
}

</style>