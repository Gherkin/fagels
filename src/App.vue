<script setup>
import Menu from './components/Menu.vue'
import axios from 'axios'
</script>


<template>
  <header>

  </header>

  <main>
    <div id="img-container">
      <v-img id="image" @click="imageClick" :key="n" v-if="n"
        :src="bilder[n][k]"
      ></v-img>
      <div id="overlay">
        <Menu :data="taxons" :key="taxons"></Menu>
        <div id="overlay-text-bg" @click.self="imageClick" ref="overlay">
          <div :key="name" class="text">
            {{name}}
          </div>
        </div> 
      </div>
    </div>
  </main>
</template>

<script>

export default {
  name: 'App',
  components: { 
    Menu
  },
  data() {
    return ({
      bilder: undefined,
      n: undefined,
      k: undefined,
      name: undefined,
      taxons: undefined,
      clicked: false
    })
  },
  methods: {
    imageClick(event) {
      console.log(event.target.id);
      console.log('click!');
      if(this.clicked) {
        this.$refs.overlay.style.opacity = 0;
        this.randomizeBird();
        this.clicked = false;
      } else {
        this.clicked = true;
        this.$refs.overlay.style.opacity = 1;
      }

    },
    randomizeBird() {
      console.log('randomizing')
      this.n = Object.keys(this.bilder)[Math.floor(Math.random() * Object.keys(this.bilder).length)];
      this.k = Math.floor(Math.random() * this.bilder[this.n].length);
     // this.n = 100116;
     // this.k = 10;
      console.log(this.n)
      console.log(this.k)
      if(!(this.bilder[this.n][this.k].endsWith('jpg') || this.bilder[this.n][this.k].endsWith('jpeg'))) {
        console.log('not jpeg!');
        console.log(this.bilder[this.n][this.k]);
        this.randomizeBird();
      }
      this.name = this.taxons[this.n];
    },
    loadData() {
      console.log('onCreate');
       axios.get("./bilder.json").then((response) => {
        this.bilder = response.data;
        //console.log(Object.keys(this.bilder))
       
        console.log(response.data)
        console.log(this.bilder[103046][0])

        axios.get("./taxons.json").then((response) => {
          this.taxons = response.data;
          this.randomizeBird();
        });
      });
    }
  },
  created() {
    this.loadData();
  }
}

</script>



<style scoped>
#img-container {
  width: 100vw;
  height: 100vh;
  margin: auto;
  display: inline-block;
}

#image {
  display: block;
  -webkit-user-select: none; /* Safari */        
  -moz-user-select: none; /* Firefox */
  -ms-user-select: none; /* IE10+/Edge */
  user-select: none; /* Standard */
}

#overlay {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  height: 100%;
  width: 100%;
  z-index: 10;
  overflow: hidden;
}

#overlay-text-bg {
  background-color: rgba(193,219,227,0.5);
  height: 100%;
  width: 100%;
  opacity: 0;
}
#overlay-text-bg .text {
  color: #373737;
  font-size: 50px;
  position: absolute;
  top: 50%;
  left: 50%;
  -webkit-transform: translate(-50%, -50%);
  -ms-transform: translate(-50%, -50%);
  transform: translate(-50%, -50%);
  text-align: center;
  -webkit-user-select: none; /* Safari */        
  -moz-user-select: none; /* Firefox */
  -ms-user-select: none; /* IE10+/Edge */
  user-select: none; /* Standard */
}

header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
