<template>
  <div class="container">
    <div>
      <nav>
        <div class="header">
          <h3>Buddy</h3>
        </div>
      </nav>
    </div>
    <MainPage />


  </div>
</template>

<script>
// @ is an alias to /src

import MainPage from "../components/MainPage";

export default {
  name: 'Home',
  components: {
    MainPage

  }
}
</script>
<style scoped lang="scss">
nav {
  background: #ffffff;
  box-shadow: 0px 2px 5px rgba(0, 58, 78, 0.15);
  height: 70px;
  display: flex;
}
.header {
  display: flex;
  flex-direction: row;
  align-items: center;
  padding: 0px 50px;
}
.header h3 {
  color: #1f2a53;
  font-weight: 600;
}
</style>
