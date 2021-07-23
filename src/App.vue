<template>
  <div id="app">
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
        <div class="iFrame" v-if="this.iFrame">
          <iFrame width="100%" height="750px"  v-bind:src="this.iFrame"></iFrame>
          <button v-on:click="this.iFrame = false">Return to Results</button>
        </div>
        <ul v-if="!this.iFrame">
          <li  v-for="book in books" :key="book.id">
            <div class="book-result" v-on:click="fetchReviews(book.id)">
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
        <small v-if="this.page >= 1 && !this.iFrame">Page: {{page}}</small>
        <div v-if="this.books && !this.iFrame" class="page-buttons">
          <button v-on:click="fetchPage" v-if="this.page >= 2" value="prev">Prev</button>
          <button v-on:click="fetchPage" v-if="this.page >= 1" value="next">Next</button>
        </div>
      </div>
        <div v-else-if="this.books && this.books.length === 0">
          <strong>No Results...</strong>
        </div>
        <div v-else-if="this.page > 0 && !this.books">
          <small>Loading...</small>
        </div>
        <div v-else>
          <strong>Search books to read reviews!</strong>
        </div>
    </main>
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
      iFrame: false
    }
  },
  methods: {
    fetchBooks (e) {
      const queryJoined = this.query.split(' ').join('+')
      if (e.key == "Enter"){
      this.iFrame = false
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
    fetchPage (e) {
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
      console.log(this.books)
    },
    fetchReviews (id) {
      console.log(id)
      fetch(`${this.base_url}/books/reviews/?id=${id}`)
      .then(r => r.json())
      .then(data => this.iFrame = data)
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
  cursor: pointer;
  border-radius: 16px 0px 16px 0px;
  transition: all .2s ease-in-out; 
}

.book-result:hover{
  transform: scale(1.1); 
}


ul {
    list-style-type: none;
}

.book-info {
  display: flex;
  flex-direction: column;
  width: 75%
}

.book-title{
  font-weight: bold;
}

.page-buttons{
  display: flex;
  justify-content: center;
  margin-top: 1%;
  margin-left: auto;
  margin-right: auto;
  width: 10%;
}
  

button {
	color: #fff !important;
	text-transform: uppercase;
	text-decoration: none;
	background: #C700A0;
  border-color: #C700A0;
	padding: 20px;
	border-radius: 5px;
	display: inline-block;
	border: none;
	transition: all 0.4s ease 0s;
  cursor: pointer;
}

.iFrame {
  display: flex;
  width: 50%;
  margin: auto;
  flex-direction: column;
}

strong {
  font-size: 20px;
}

</style>
