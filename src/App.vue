<template>
  <div id="app">
    <img alt="Voice123 logo" class="logo" src="./assets/logo-branded-voice123.png">
    <section class="form">
      <form @submit.prevent="runQuery">
        <label for="search">Text to search</label>
        <input type="search" if="search" v-model="keywords" value="test">
        <button type="submit">Search</button>
      </form>
    </section>
    <section class="profiles">
      <article class="profiles__item" v-for="profile in profiles" :key="profile.id">
        <div class="profiles__item__image">
          <a :href="`https://voice123.com/${profile.user.username}`">
            <img :src="profile.user.picture_large" :alt="profile.user.name"
                  v-if="profile.user.picture_large">
            <img src="./assets/500.png" :alt="`Picture missing - ${profile.user.name}`" v-else>
          </a>
        </div>
        <div class="profiles__item__content">
          <a :href="`https://voice123.com/${profile.user.username}`">
            <h3>{{ profile.user.name }}</h3>
          </a>
          <p>{{ profile.summary | maxLength }}</p>
          <audio :src="`https://api.sandbox.voice123.com/${profile.relevant_sample.file}`">
            Your browser does not support the <code>audio</code> element.
          </audio>
        </div>
      </article>
    </section>
    <Pagination :current-page="current_page" :total-pages="total_pages"
                :per-page="page_size" :total="total_rows" @pagechanged="onPageChange"
                :url="url" :keywords="keywords" v-if="total_pages > 0" />
  </div>
</template>

<script>
import Pagination from './components/Pagination.vue';

const API_URL = 'https://api.sandbox.voice123.com/providers/search/?service=voice_over&keywords=';

export default {
  name: 'app',
  components: {
    Pagination,
  },
  data: () => ({
    error: '',
    current_page: 0,
    page_size: 0,
    total_pages: 0,
    total_rows: 0,
    keywords: '',
    url: API_URL,
    profiles: [],
  }),
  filters: {
    lineBreaks(value) {
      if (!value) return '';
      // eslint-disable-next-line
      value = value.toString();
      return value.replace(/(?:\r\n|\r|\n)/g, '<br>');
    },
    hightlight(value) {
      const keywords = this.keywords;
      if (!value) return '';
      if (!keywords) return value;
      // eslint-disable-next-line
      value = value.toString();
      return value.replace(keywords, '<span class="highlight">' + keywords + '</span>');
    },
    maxLength(value) {
      if (!value) return '';
      // eslint-disable-next-line
      value = value.toString().substring(0, 400) + '...';
      return value;
    },
  },
  methods: {
    runQuery(event, querypage) {
      let url = `${API_URL}${this.keywords}&page=`;
      if (querypage > 1) { url += querypage; } else { url += '1'; }
      fetch(url)
        .then((response) => {
          if (response.status !== 200) {
            const error = response.statusText;
            this.error = error;
          }
          this.current_page = parseInt(response.headers.get('x-list-current-page'), 10);
          this.page_size = parseInt(response.headers.get('x-list-page-size'), 10);
          this.total_pages = parseInt(response.headers.get('x-list-total-pages'), 10);
          this.total_rows = parseInt(response.headers.get('x-list-total-rows'), 10);
          return response.json();
        })
        .then((result) => {
          this.error = '';
          this.profiles = result.providers;
        });
    },
    onPageChange(page) { this.current_page = page; },
  },
};
</script>

<style lang="scss">
  $black: #000000;
  $white: #ffffff;
  $color-1: #285bd4;
  $color-2: #85edee;
  $color-3: #cffcf6;

  body {
    font-size: 16px;
  }

  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: rgba( $black, 0.8 );
    display: block;
    margin: 4rem auto 2rem;
    max-width: 1000px;
    img.logo {
      max-width: 25rem;
    }
  }

  form {
    display: block;
    margin: 2rem auto;
    width: 70%;
    label {
      display: none;
    }
    input[type=search] {
      background-color: $white;
      border: 1px solid $color-1;
      border-radius: 20px;
      color: $color-1;
      font-size: 1.2rem;
      padding: 0.5rem 1rem;
      width: 100%;
      transition: 0.3s ease-in;
      &:hover, &:active {
        box-shadow: 0 2px 3px rgba( $black, 0.3 );
      }
    }
    button {
      background-color: $color-1;
      border: unset;
      border-radius: 20px;
      color: $white;
      cursor: pointer;
      font-size: 1rem;
      margin: 1rem;
      padding: 0.5rem 5rem;
      transition: 0.3s ease-in;
      &:hover {
        box-shadow: 0 2px 3px rgba( $black, 0.3 );
      }
    }
  }

  .profiles {
    display: block;
    & &__item {
      display: flex;
      flex-direction: row;
      margin: 1rem 0;
      &__image {
        flex: 0 0 20%;
        img {
          max-width: 100%;
        }
      }
      &__content {
        flex: 0 0 80%;
        text-align: left;
        padding: 0 1rem;
        h3 {
          margin: 0;
          padding: 0;
        }
        a {
          color: $color-1;
          text-decoration: none;
          &:hover {
            text-decoration: underline;
          }
        }
        .highlight {
          background-color: $color-3;
        }
      }
    }
  }

  @media screen and (max-width: 1000px) {
    #app {
      margin: 4rem 1rem 2rem;
    }
  }
  @media screen and (max-width: 768px) {
    form {
      width: 90%;
    }
  }
</style>
