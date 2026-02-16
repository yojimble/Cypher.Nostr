<template>
  <div class="gallery">
    <h1>Gallery</h1>
    <div class="gallery-items">
      <div class="gallery-item" v-for="item in items" :key="item.id">
        <img :src="item.image" :alt="item.title" />
        <h2>{{ item.title }}</h2>
        <p>{{ item.description }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, image: 'path/to/image1.jpg', title: 'Image 1', description: 'Description of Image 1' },
        { id: 2, image: 'path/to/image2.jpg', title: 'Image 2', description: 'Description of Image 2' },
        { id: 3, image: 'path/to/image3.jpg', title: 'Image 3', description: 'Description of Image 3' },
      ],
    };
  },
};
</script>

<style>
.gallery {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.gallery-items {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}
.gallery-item {
  margin: 10px;
  border: 1px solid #ccc;
  padding: 10px;
}
</style>
