<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="9" md="8">
      <v-card>
        <v-card-title class="headline">
          Welcome to the RepoRanker2
        </v-card-title>
        
        <v-card-text>
          <v-text-field
            dark
            v-model="url"
            label="Please enter your github url"
            :error-messages="urlErrors"
            @change="urlErrors=null"
            @keydown.enter="getRepo"
          ></v-text-field>
          <v-btn  block :disabled="loading" @click="getRepo">GET THE REPO</v-btn>
        </v-card-text>
        <v-card-text>
          <div v-if="loading" class="pb-5"> 
            <Loader></Loader>
          </div>
          <div v-else-if="repo && repo.contributors">

             <v-list-item v-for="(contrib, index) in repo.contributors" :key="index">
                <v-list-item-icon class="my-auto">
                  {{index+1}}
                </v-list-item-icon>
                <v-list-item-icon class="my-auto">
                  <v-avatar>
                    <img
                      :src="contrib.avatarUrl"
                      :alt="contrib.id"
                    >
                  </v-avatar>
                </v-list-item-icon>
                <v-list-item-content>
                  <v-row class="pl-2">
                    <v-col cols="5" class="text-center ma-auto">
                      {{contrib.id}}
                    </v-col>
                    <v-col cols="7" class="text-right">
                      <div class="py-2"><span class="success--text">{{contrib.adds}} Additions</span> + <span class="error--text">{{contrib.dels}} Deletions</span></div>
                      <div>= {{contrib.total}} Updates</div>
                    </v-col>
                  </v-row>                  
                </v-list-item-content>
              </v-list-item>
          </div>
          
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
import isGithubUrl from 'is-github-url'
import Loader from '../components/Loader'
export default {
  data: ()=> ({
    title: 'No repo selected',
    url:null,
    loading:false,
    repo:null,
    urlErrors:null,
  }),
  methods:{
    checkUrl(url){
      this.urlErrors=null
      console.log('running check')
      if (!url){
        this.urlErrors="Please enter an url"
        return false
      }
      if (!isGithubUrl(url)){
        this.urlErrors="Please enter a VALID GITHUB url"
        return false
      }
      return true
    },
    getRepo(){
      this.loading=true

      if (!this.checkUrl(this.url)){
        this.loading=false
        return
      }
      let vm=this
      let config = {
        method: 'post',
        url: `https://api-github11sigma.herokuapp.com/graphql?query=query{\n  repo(url:"${vm.url}") {\n   owner,repo_name,url,contributors {\n      id, url, avatarUrl,contributions, adds, dels, commits, total\n    }\n  }\n}`,
        headers: { 
          'Bearer': process.env.GITHUB_KEY
        }
      };
      this.$axios(config)
      .then(res => {
        console.log(res.data.data.repo)
        if (res.data.data.repo.repo_name==null){
          vm.urlErrors="Cant find this repo please check your url"
        }
        vm.repo=res.data.data.repo
        if (vm.repo && vm.repo.contributors){
          vm.repo.contributors.sort((a,b)=>{
            if (a.total > b.total) return -1;
            if (a.total < b.total) return 1;
            return 0;
          })
        }
        vm.loading=false
      })
      .catch(function (error) {
        console.log(error)
        vm.loading=false
      });

    }
  },
  components:{
    Loader
  }
}
</script>
