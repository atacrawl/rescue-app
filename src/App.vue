<template>
  <div id="app">
    <header>
      <img alt="Find a Dog logo" src="./assets/dog-api-logo.svg">
      <h1>Find a Dog</h1>
    </header>
    <section>
      <input type="text" @keyup="clearImages" v-model="dogString">
      <button class="search-dogs" @click="fetchBreeds">Go</button>
      <div class="results" v-if="foundBreeds">
        <h4>Results</h4>
        <ul>
          <li v-for="breed in foundBreeds" :key="breed">
            <button @click="fetchImages(breed)">{{ breed | titleCase }}</button>
          </li>
        </ul>
      </div>
      <p class="error" v-if="errorText">{{ errorText }}</p>
      <div class="modal" v-if="imagesList">
        <button class="close-modal" @click="clearImages">X</button>
        <h4>Results</h4>
        <h2>{{ selectedBreed | titleCase }}</h2>
        <carousel
          :per-page="1"
          :pagination-active-color="'#BF3636'"
          :pagination-padding="5">
          <slide v-for="(img, i) in imagesList" :key="i">
            <div class="slide">
              <img :src="img" :alt="`Image ${i + 1}`">
            </div>
          </slide>
        </carousel>
      </div>
    </section>
  </div>
</template>

<script>
import axios from 'axios';
import { Carousel, Slide } from 'vue-carousel';

export default {
  name: 'App',
  components: {
    Carousel,
    Slide,
  },
  data() {
    return {
      allBreeds: [],
      imagesList: null,
      dogString: '',
      selectedBreed: '',
      foundBreeds: null,
      errorText: null,
    };
  },
  filters: {
    titleCase(breed) {
      const splitBreed = breed.split(' ');
      for (let x = 0; x < splitBreed.length; x++) {
        splitBreed[x] = splitBreed[x].charAt(0).toUpperCase() + splitBreed[x].slice(1);
      }
      return splitBreed.join(' ');
    },
  },
  methods: {
    clearImages() {
      this.imagesList = null;
      this.errorText = null;
    },
    clearAll() {
      this.clearResults();
      this.clearImages();
    },
    fetchBreeds() {
      if (this.dogString === '') {
        this.errorText = 'Error: No search entered';
        this.foundBreeds = null;
      } else {
        this.foundBreeds = this.allBreeds.filter(breed => breed.indexOf(this.dogString) !== -1);
        if (this.foundBreeds.length === 0) {
          this.errorText = 'Error: No dogs found';
          this.foundBreeds = null;
        }
      }
    },
    fetchImages(dog) {
      this.selectedBreed = dog;
      const splitDog = dog.split(' ');
      const fixedDog = splitDog[1] ? `${splitDog[1]}/${splitDog[0]}` : dog;
      axios
        .get(`https://dog.ceo/api/breed/${fixedDog}/images/random/3`)
        .then(response => (this.imagesList = response.data.message))
        .catch(error => (console.log(error)));
    },
    standardizeBreeds(data) {
      Object.entries(data).map(row => {
        const rootBreed = row[0];
        const subBreed = row[1];
        if (subBreed.length > 0) {
          subBreed.forEach(sub => {
            const combinedBreed = `${sub} ${rootBreed}`;
            this.allBreeds.push(combinedBreed.toLowerCase());
          });
        } else {
          this.allBreeds.push(rootBreed.toLowerCase());
        }
        return false;
      });
    },
  },
  mounted() {
    axios
      .get('https://dog.ceo/api/breeds/list/all')
      .then(response => this.standardizeBreeds(response.data.message))
      .catch(error => (console.log(error)));
  },
};
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;900&display=swap');

*,
*::before,
*::after {
  margin: 0;
  box-sizing: border-box;
}

:root {
  --gradient-top: #00A1FC;
  --gradient-bottom: #054C74;
  --red: #BF3636;
  --yellow: #FFCB05;
}

html {
  font-size: 10px;
}

body {
  font: 1em/1.2 'Roboto', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: white;
  background: linear-gradient(180deg, var(--gradient-top) 0%, var(--gradient-bottom) 100%) fixed;
}

img {
  max-width: 100%;
  display: block;
}

button {
  appearance: none;
  border: none;
  background: transparent;
  font: inherit;
  color: inherit;
  cursor: pointer;
}

#app {
  margin: 0 auto;
  width: 375px;
}

h1,
h2,
h4,
ul,
.error,
.close-modal {
  font-size: 2rem;
  font-weight: 900;
}

h2,
ul {
  font-weight: 400;
}

header {
  text-align: center;
  padding-top: 2rem;

  img {
    margin: 0 auto 1rem;
    width: 10.7rem;
    height: auto;
  }
}

input {
  display: block;
  border: none;
  width: 280px;
  margin: 20px auto;
  font-weight: 900;
  font-size: 50px;
  line-height: 1;
  padding: 5px;
  box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25);
  border-radius: 10px;
}

.search-dogs {
  display: block;
  margin: 0 auto;
  background: var(--yellow);
  border-radius: 30px;
  color: var(--red);
  text-transform: uppercase;
  font-size: 3rem;
  line-height: 2;
  font-weight: 900;
  width: 100px;
  box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25);
}

.error {
  margin-top: 35px;
  text-align: center;
}

.results {
  margin: 35px auto;
  width: 280px;

  h4 {
    text-align: center;
    padding-bottom: 30px;
  }

  ul {
    li:not(:last-child) {
      padding-bottom: 1em;
    }

    button {
      color: white;
      text-decoration: underline;
      text-align: left;
    }
  }
}

.modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 300px;
  height: 80vh;
  max-height: 500px;
  overflow: hidden;
  background: var(--yellow);
  color: var(--red);

  .close-modal {
    position: absolute;
    top: 0;
    right: 0;
  }

  h4 {
    padding: 40px 20px;
  }

  h2 {
    text-align: center;
    padding-bottom: 10px;
  }
}

.slide {
  padding: 0 10px;
  width: 300px;
  height: 300px;
  display: flex;
  align-items: center;
  justify-content: center;

  img {
    max-width: 100%;
    max-height: 100%;
  }
}
</style>
