<template>
  <div v-if="!isLoading">
    <TopHeader/>
    <section class="section">
      <div class="container">
        <div v-for="sp in project.subProjects" :key="sp.id" class="subproject">
          <p class="title is-size-4">{{ sp.name }}</p>
          <Description :description="sp.description" />
          <div class="button-container">
            <SoundButton
              v-for="item in sp.items"
              :key="item.id"
              :itemData="item"
            />
          </div>
        </div>
      </div>
    </section>

    <!-- <section class="section" v-if="project.settings.toneOptions.showFx">
      <div class="container">
        <PlayerMasterController />
      </div>
    </section> -->
  </div>
</template>

<script>
import { mapState, mapGetters } from "vuex";

export default {
  head() {
    return {
      title: `${this.project.name} | Plan8`,
    };
  },
  data() {
    return {
      // project: {
      //     name: ''
      // },
    };
  },
  computed: {
    ...mapState(["isLoading", "project"]),
  },
  mounted() {
    console.log(this.project);
    this.$store.commit("SET_ISLOADING", false);
  },

  async asyncData({ route, store, app }) {
    try {
      const { id } = route.params;
      console.log("id: ", id);

      const apiRoot = process.env.NODE_ENV == 'production' ? 'https://hqapi.plan8.co/v1' : 'http://localhost:3000/v1';


      let projectId;

      if (id) {
        projectId = id;
      } else {
        projectId = "619b7749fc984a00176ee238";
      }
      const projectPath = `${apiRoot}/projects/${projectId}/public`;
      const project = await $fetch(projectPath);

      // store.commit('SET_ISLOADING', false)

      store.commit("SET_PROJECT", project);

      store.commit('player/SET_USETONE', true)

      // return {
      //   project,
      // };
    } catch (error) {
      console.log("error!: ", error);

      return {
        // project: {
        //   name: 'Not Found'
        // }
      };
    }
  },
};
</script>

<style lang="scss">

</style>
