<template>
  <div class="about">
    <div>
    Created by: <a :href="myWebsiteHttps">{{ myWebsite }}</a>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src

export default {
  name: 'About',
  components: {
  },
  data() {
    return {
      emailLocal: 'tim',
      domain: 'timoslund',
    };
  },
  computed: {
    myWebsite() {
      return `www.${this.domain}.com`;
    },
    myWebsiteHttps() {
      return `https://www.${this.domain}.com`;
    },
  },
};
</script>
