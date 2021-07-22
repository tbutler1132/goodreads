<template>
  <div class="book">
    <p>{{oh}}</p>
  </div>
</template>

<script>

export default {
  name: 'Book',
  props:{
      oh: String
  }
}

</script>