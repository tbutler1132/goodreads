<template>
  <div class="app">
    <div v-if="this.home" class="not-login">   
      <nav>
        <p v-if="this.user">{{this.user.username}}</p>
        <p v-else>Welcome</p>
        <i class="fa fa-home" v-on:click="this.page = 0"></i>
        <p v-if="!this.user" v-on:click="this.home = false">Login</p>
        <p v-else v-on:click="logoutUser">Logout</p>
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
              <div class="book-result">
                  <div class="book-cover">
                    <img v-bind:src="book.image" />
                  </div>
                <div class="book-info">
                  <span class="book-title">{{ book.title }}</span>
                    <span> by {{book.author}}</span>
                </div>
              <p v-on:click="addBook(book.title, book.author, book.image)">+</p>
              </div>
            </li>
          </ul>
          <span v-if="this.page >= 1">Page: {{page}}</span>
          <div v-if="this.books" class="page-buttons">
            <button class="page" v-on:click="fetchNextPage" v-if="this.page >= 2" value="prev">Prev</button>
            <button class="page" v-on:click="fetchNextPage" v-if="this.page >= 1" value="next">Next</button>
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
            <ul v-if="this.user.books.length > 0">
              <li v-for="book in this.user.books" :key="book._id">
                <div class="book-result">
                  <div class="book-cover">
                    <img v-bind:src="book.image" />
                  </div>
                  <div class="book-info">
                    <span class="book-title">{{ book.title }}</span>
                    <span> by {{book.author}}</span>
                  </div>
                <p v-on:click="deleteBook(book._id)">-</p>
                </div>
              </li>
            </ul>
            <span v-else>Search books to add to your reading list!</span>
          </div>
        </div>
      </main>
    </div>
    <div v-else class="user-form">
      <form v-if="!this.newUser" v-on:submit="(e) => signinUser(e, 'signin')">
        <label>Username</label>
        <input v-model="this.username" type="text" />
        <label>Password</label>
        <input v-model="this.password" type="password" />
        <button type="submit">Signin</button>
        <p v-on:click="this.newUser = true">Not a user? Signup!</p>
      </form>
      <form v-else v-on:submit="(e) => signinUser(e, 'signup')">
        <label>Username</label>
        <input v-model="this.username" type="text" />
        <label>Password</label>
        <input v-model="this.password" type="password" />
        <button id="sign-in" class="page" type="submit">Signup</button>
        <p v-on:click="this.newUser = false">Already a user? Sign in!</p>
      </form>
    </div>
  </div>
</template>

<script>

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
      newUser: true,
    }
  },
  async created () {
    const token = localStorage.getItem('token')
    const profile = localStorage.getItem("profile")
    if(token){
        const profileObj = JSON.parse(profile)
        fetch(`${this.base_url}/users/${profileObj._id}`, {
        method: 'GET',
        headers: {
          'Authorization': `Bearer ${token}`,
          'Content-Type': 'application/json',
          'Accept': 'application/json'
        }
      })
      .then(r => r.json())
      .then(data=> {
        this.user = data
      })
    }
  },
  methods: {
    fetchBooks (e) {
      const queryJoined = this.query.split(' ').join('+')
      if (e.key == "Enter"){
        this.page = 1
        fetch(`${this.base_url}/books/search?term=${queryJoined}&page=${this.page}`)
        .then(r => r.json())
        .then(data => {
          this.setBooks(data)})
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
        fetch(`${this.base_url}/books/search?term=${this.query}&page=${this.page}`)
        .then(r => r.json())
        .then(data => this.setBooks(data))
        .catch(err => {
          console.log(err)
          this.page = 0
          this.books = false
          alert("Something went wrong")})
          document.documentElement.scrollTop = 0
    },
    setBooks(books) {
      this.books = books
    },
    signinUser(event, type){
      event.preventDefault()
      const options = {
        method: 'post',
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json'
        },
        body: JSON.stringify({
          username: this.username,
          password: this.password
        })
      }
      fetch(`${this.base_url}/users/${type}`, options)
      .then(r => r.json())
      .then(data => {
        console.log(data)
        this.user = data.result, 
        this.home = true
        localStorage.setItem("token", data.token)
        localStorage.setItem("profile", JSON.stringify({...data.result}))
      })
    },
    logoutUser (){
      localStorage.removeItem("token")
      localStorage.removeItem("profile")
      this.user = false
    },
    async addBook (title, author, image) {
      console.log(title, author, image)
      const options = {
        method: 'post',
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json'
        },
        body: JSON.stringify({
          title: title,
          author: author,
          image: image
        })
      }
      fetch(`${this.base_url}/users/${this.user._id}/books`, options)
      .then(r => r.json())
      .then(data => this.user = data)
    },
    async deleteBook (bookId) {
      const options = {
        method: 'DELETE'
      }
      fetch(`${this.base_url}/users/${this.user._id}/books/${bookId}`, options)
      .then(r => r.json())
      .then(data => this.user = data)
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
  margin-top: 4%;
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

button.page {
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

#sign-in{
  width: 50%;
  border: solid;
  margin-left: 25%;
  margin-right: 25%;
}

.my-button {
    color: #333;
    background-color: #fff;
    border-color: #ccc;
}
.my-button:focus {
    color: #333;
    background-color: #e6e6e6;
    border-color: #8c8c8c;
}
.my-button:hover {
    color: #333;
    background-color: #e6e6e6;
    border-color: #adadad;
}
.my-button:active {
    color: #333;
    background-color: #e6e6e6;
    border-color: #adadad;
}

nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: auto;
  width: 25%;
  float: center;
  transition: all .2s ease-in-out; 
}


nav > p {
  cursor: pointer;
  transition: all .2s ease-in-out; 
}

nav > p:hover{
  transform: scale(1.5); 
}

i {
  cursor: pointer;
  transition: all .2s ease-in-out; 
}

i:hover{
  transform: scale(1.8); 
}

form{
  display: flex;
  flex-direction: column;
  width: 20%;
  margin: auto;
  justify-content: space-around;
  height: 300px;
}
</style>
