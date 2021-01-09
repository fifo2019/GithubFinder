<template>
  <div class="wrapper-content wrapper-content--fixed">
    <section>
      <div class="container">
  
        <!-- errors -->
        <div class="error" v-if="error" style="margin-bottom: 20px;">
          <p>{{ error }}</p>
        </div>
        
        <!-- search-->
        <search
          :value="search"
          placeholder="Type username..."
          @search="search = $event"
        />
  
        <!-- buttons search -->
        <button
          v-if="!repos"
          @click="getRepos"
          class="btn btnPrimary"
        >
          Search!
        </button>
        <button
          v-else
          @click="getRepos"
          class="btn btnPrimary"
        >
          Search Again
          !
        </button>
  
        <!-- wrapper repos -->
        <div class="repos__wrapper" v-if="repos">
  
          <div class="user-profile">
            
            <ul>
              <li>
                <img
                  width="300"
                  height="300"
                  :src="user.avatar_url"
                  :alt="user.name"
                  style="object-fit: cover;"
                />
              </li>
              <li>
                <p>
                  Username: {{ user.name || 'No name' }}
                </p>
              </li>
              <li>
                <p>
                  Count repos: {{ user.public_repos }}
                </p>
              </li>
            </ul>
          </div>
          
          <!-- item -->
          <table>
            <thead>
              <tr>
                <td @click="sort('name')">Repo name</td>
                <td @click="sort('stargazers_count')">Stargazers</td>
              </tr>
            </thead>
            
            <tbody>
              <tr
                v-for="repo in reposSort"
                :key="repo.id"
              >
                <td>
                  <a
                    :href="repo.html_url"
                    target="_blank"
                    class="link"
                  >
                    {{ repo.name }}
                  </a>
                </td>
                <td style="text-align: center;">
                  <span>{{ repo.stargazers_count }} ⭐</span>
                </td>
              </tr>
            </tbody>
          </table>
  
          <!-- buttons -->
          <section>
            <div class="contaier">
              <div class="button-list">
                <div class="btn btnPrimary" @click="prevPage"> &#8592; </div>
                <div class="btn btnPrimary" @click="nextPage"> &#8594; </div>
              </div>
            </div>
          </section>
          
        </div>
        
      </div>
    </section>
  </div>
</template>

<script>
  import axios from 'axios'
  
  import search from '@/components/Search.vue'
  
  export default {
    name: "Home",
    components: {
      search
    },
    data() {
      return {
        search: '',
        error: null,
        repos: null,
        user: null,
        currentSort: 'name',
        currentSortDir: 'asc',
        page: {
          current: 1,
          length: 6
        }
      }
    },
    computed: {
      reposSort () {
        
        return this.repos.sort((a, b) => {
          let mod = 1
          if (this.currentSortDir === 'desc') mod = -1
          if (a[this.currentSort] < b[this.currentSort]) return -1 * mod
          if (a[this.currentSort] > b[this.currentSort]) return 1 * mod
          return 0
        }).filter((row, index) => {
          let start = (this.page.current-1)*this.page.length
          let end = this.page.current * this.page.length
          if (index >= start && index < end) return true
        })
      },
    },
    methods: {
      getRepos() {
        axios
          .get(`https://api.github.com/users/${this.search}`)
          .then(res=> {

            axios
              .get(`https://api.github.com/users/${this.search}/repos`)
              .then(res=> {
                this.repos = res.data
              })
              .catch(err => console.log(err))
            
            this.user = res.data
            this.error = null
          })
          .catch(err => {
            this.user = null
            this.repos = null
            this.error = 'Can`t find this user'
          })
      },
      sort (e) {
        if (e === this.currentSort) {
          this.currentSortDir = this.currentSortDir === 'asc' ? 'desc' : 'asc'
        }
        this.currentSort = e
      },
      // Pagination
      prevPage () {
        if (this.page.current > 1) this.page.current-=1
      },
      nextPage () {
        if ((this.page.current * this.page.length) < this.repos.length) this.page.current+=1
      }
    }
  }
</script>

<style lang="scss" scoped>
  .container {
    display: flex;
    align-items: center;
    flex-direction: column;
  }
  
  button {
    margin: 40px 0 0 0;
  }
  
  .repos__wrapper {
    width: 500px;
    margin: 30px 0;
  }
  
  .repos-info {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 10px;
    padding: 10px 0;
    border-bottom: 1px solid #dbdbdb;
  }
  
  .user-profile {
    display: flex;
    justify-content: center;
  }
  
  ul li {
    margin-bottom: 10px;
  }

  thead td {
    cursor: pointer;
  }
  
  .button-list {
    width: 100%;
    display: flex;
    justify-content: space-between;
  }
</style>