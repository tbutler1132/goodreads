<template>
  <div class="app">
    <div v-if="this.home" class="not-login">   
      <nav>
        <button v-on:click="this.page = 0">home</button>
        <button v-on:click="this.home = false">Login</button>
      </nav>
      <main>
        <div class="search-box" >
          <input
          class="search-bar" 
          type="text" 
          placeholder="search"
          v-model="query"
          @keypress="fetchBooks"
          />
        </div>
        <div v-if="this.books && this.books.length > 0">
          <ul>
            <li v-for="book in books" :key="book.id">
              <p v-on:click="addBook(book.title, book.author, book.image)">+</p>
              <div class="book-result">
                  <div class="book-cover">
                    <img v-bind:src="book.image" />
                  </div>
                <div class="book-info">
                  <span class="book-title">{{ book.title }}</span>
                    <span> by {{book.author}}</span>
                </div>
              </div>
            </li>
          </ul>
          <span v-if="this.page >= 1">Page: {{page}}</span>
          <div v-if="this.books" class="page-buttons">
            <button v-on:click="fetchNextPage" v-if="this.page >= 2" value="prev">Prev</button>
            <button v-on:click="fetchNextPage" v-if="this.page >= 1" value="next">Next</button>
          </div>
        </div>
        <div v-else-if="this.books && this.books.length === 0">
          <span>No more books to display!</span>
        </div>
        <div v-else-if="this.page > 0 && !this.books">
          <span>Loading...</span>
        </div>
        <div v-else>
          <span v-if="!this.user">Search for a book!</span>
          <div v-else class="user-books">
          <ul>
            <li v-for="book in this.user.books" :key="book._id">
              <p v-on:click="deleteBook(book._id)">-</p>
              <div class="book-result">
                <div class="book-cover">
                  <img v-bind:src="book.image" />
                </div>
                <div class="book-info">
                  <span class="book-title">{{ book.title }}</span>
                  <span> by {{book.author}}</span>
                </div>
              </div>
            </li>
          </ul>
          </div>
        </div>
      </main>
    </div>
    <div v-else class="login">
      <form v-on:submit="signinUser">
        <label>Username</label>
        <input v-model="this.username" type="text" />
        <label>Password</label>
        <input v-model="this.password" type="password" />
        <button type="submit">Signin</button>
      </form>
    </div>
  </div>
</template>

<script>

import axios from 'axios'


export default {
  name: 'App',
  data () {
    return {
      base_url: 'http://localhost:7000',
      query: '',
      page: 0,
      books: false,
      home: true,
      username: '',
      password: '',
      user: false,
    }
  },
  async created () {
    const token = localStorage.getItem('token')
    const profile = localStorage.getItem("profile")
    if(token){
        const profileObj = JSON.parse(profile)
        axios({
        method: 'GET',
        url: `${this.base_url}/users/${profileObj._id}`,
        headers: {
          'Authorization': `Bearer ${token}`,
          'Content-Type': 'application/json',
          'Accept': 'application/json'
        }
      })
      .then(data=> {
        this.user = data.data
      })
    }
  },
  methods: {
    fetchBooks (e) {
      const queryJoined = this.query.split(' ').join('+')
      if (e.key == "Enter"){
        this.page = 1
        axios(`${this.base_url}/books/search?term=${queryJoined}&page=${this.page}`)
        .then(data => {
          console.log(data.data)
          this.setBooks(data.data)})
        .catch(err => {
          console.log(err)
          this.page = 0
          alert("Something went wrong")})
      }
    },
    fetchNextPage (e) {
      this.books = false
      if(e.target.value === "next"){
        this.page = this.page + 1
      }else{
        this.page = this.page - 1
      }
        axios(`${this.base_url}/books/search?term=${this.query}&page=${this.page}`)
        .then(data => this.setBooks(data.data))
        .catch(err => {
          console.log(err)
          this.page = 0
          this.books = false
          alert("Something went wrong")})
          document.documentElement.scrollTop = 0
    },
    setBooks(books) {
      this.books = books
      console.log(this.books)
    },
    signinUser (e) {
      e.preventDefault()
      const options = {
        method: 'post',
        url: `${this.base_url}/users/signin`,
        data: {
          username: this.username,
          password: this.password
        }
      }
      axios(options)
      .then(data => {
        this.user = data.data.result, 
        this.home = true
        console.log(data)
        console.log(this.user)
        localStorage.setItem("token", data.data.token)
        localStorage.setItem("profile", JSON.stringify({...data.data.result}))
      })
    },
    async addBook (title, author, image) {
      console.log(title, author, image)
      const user = await axios.post(`${this.base_url}/users/${this.user._id}/books`, {
        title: title,
        author: author,
        image: image
      })
      this.user = user
    },
    async deleteBook (bookId) {
      const user = await axios.delete(`${this.base_url}/users/${this.user._id}/books/${bookId}`)
      this.user = user
    }
  }

}



</script>

<style>

body {
background: rgb(131,58,180);
background: linear-gradient(90deg, rgba(131,58,180,1) 0%, rgba(253,29,29,1) 50%, rgba(168,69,252,1) 100%);
scroll-behavior: smooth;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  margin-bottom: 60px;
}

.search-box {
  width: 100%;
  margin-bottom: 30px;
  display: flex;
  justify-content: center;
}
.search-box .search-bar {
  display: flex;
  width: 30%;
  padding: 15px;
  color: #313131;
  font-size: 20px;
  appearance: none;
  border:none;
  outline: none;
  background: none;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}
.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}

.book-result{
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin-bottom: 2%;
  margin-left: 24%;
  margin-right: 24%;
  padding-bottom: 1%;
  padding-top: 1%;
  background-color: #4cc9f0;

  border-radius: 16px 0px 16px 0px;
}

ul {
    list-style-type: none;
}

.book-info {
  display: flex;
  flex-direction: column;
  width: 75%
}

.book-cover {

}

.book-title{
  font-weight: bold;
}

.page-buttons{
  display: flex;
  justify-content: center
}

button {
	color: #fff !important;
	text-transform: uppercase;
	text-decoration: none;
	background: #ed3330;
	padding: 20px;
	border-radius: 5px;
	display: inline-block;
	border: none;
	transition: all 0.4s ease 0s;
  cursor: pointer;
}

nav {
  width: 25%;
  float: right
}
</style>
