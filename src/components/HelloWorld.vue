

<template>
  <div class="hello">
    <main>
      <div className="search-box" >
        <p>{{msg}}</p>
        <input 
        type="text" 
        placeholder="search"
        v-model="query"
        @keypress="fetchBooks"
        />
        <p v-if="books[0] !== undefined">{{ books[0].title[0] }}</p>
      </div>
    </main>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Search',
  data () {
    return {
      base_url: 'http://localhost:7000/books/search',
      query: '',
      books: { }
    }
  },
  props: {
    msg: String,
  },
  methods: {
    fetchBooks (e) {
      if (e.key == "Enter"){
        console.log(this.query)
        axios(`${this.base_url}?term=${this.query}&page=1`)
        .then(data => this.setBooks(data.data))
      }
    },
    setBooks(books) {
      this.books = books
      console.log(this.books)
    }
  }
}


</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
