<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <h1>{{ title }}</h1>
     <v-select label="name" :filterable="false"
      @close="onClose"
      :options="options" @search="fetchOptions">
      <template slot="no-options">
        type to search GitHub repositories..
      </template>
      <template slot="option" slot-scope="option">
        <div class="d-center">
          <img :src="option.owner.avatar_url" />
          {{ option.full_name }}
        </div>
      </template>
      <template slot="selected-option" slot-scope="option">
        <div class="selected d-center">
          <img :src="option.owner.avatar_url" />
          {{ option.full_name }}
        </div>
      </template>
       <template #list-footer>
          <li ref="load" class="loader" v-show="hasNextPage">
            Loading more options...
          </li>
        </template>
    </v-select>
  </div>
</template>

<script>
import 'vue-select/dist/vue-select.css';
import vSelect from 'vue-select';
import _ from 'lodash';

export default {
  name: 'App',
  components: {
    vSelect,
  },
  data() {
    return {
      title: 'Select 2 Infinite search',
      options: [],
      observer: null,
      page: 0,
      keyword: ''
    };
  },
   mounted () {
    /**
     * You could do this directly in data(), but since these docs
     * are server side rendered, IntersectionObserver doesn't exist
     * in that environment, so we need to do it in mounted() instead.
     */
    this.observer = new IntersectionObserver(this.infiniteScroll);
  },
  methods: {
    fetchOptions(search, loading) {
      if (search.length) {
        this.keyword = search;
        loading(true);
        this.search(loading, search, this);
      }
    },
    hasNextPage: () => {
      return false;
    },

    search: _.debounce((loading, search, vm) => {
      fetch(
        `https://api.github.com/search/repositories?q=${escape(search)}`
      ).then((res) => {
        res.json().then((json) => (vm.options = json.items));
        loading(false);
        setTimeout(async () => {
          console.log(vm.$refs.load);
          await vm.$nextTick();
          vm.observer.observe(vm.$refs.load);
        }, 400);
      });
    }, 350),
    fetchPagedData: (search, page, vm) => {
      fetch(
        `https://api.github.com/search/repositories?q=${escape(search)}&page=${page}`
      ).then((res) => {
        res.json().then((json) => (vm.options = json.items));
      });
    },
    onClose () {
      this.observer.disconnect();
    },

    async infiniteScroll ([{isIntersecting, target}]) {
      console.log(isIntersecting, target);
      if (isIntersecting) {
        this.page = this.page + 1;
        const ul = target.offsetParent;
        const scrollTop = target.offsetParent.scrollTop;
        this.limit += 10;
        await this.$nextTick();
        ul.scrollTop = scrollTop;
        this.fetchPagedData(this.keyword, this.page, this);
      }
    }
  },
};
</script>

<style>
img {
  height: auto;
  max-width: 2.5rem;
  margin-right: 1rem;
}

.d-center {
  display: flex;
  align-items: center;
}

.selected img {
  width: auto;
  max-height: 23px;
  margin-right: 0.5rem;
}

.v-select .dropdown li {
  border-bottom: 1px solid rgba(112, 128, 144, 0.1);
}

.v-select .dropdown li:last-child {
  border-bottom: none;
}

.v-select .dropdown li a {
  padding: 10px 20px;
  width: 100%;
  font-size: 1.25em;
  color: #3c3c3c;
}

.v-select .dropdown-menu .active > a {
  color: #fff;
}


</style>
