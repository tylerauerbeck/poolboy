<template>
  <h1>ResourceHandle {{$route.params.name}}</h1>
  <p>{{ error }}</p>
  <p v-if="resourcePoolRef"><router-link :to="'/resourcepool/' + resourcePoolRef.namespace + '/' + resourcePoolRef.name">ResourcePool {{resourcePoolRef.name}}</router-link></p>
  <p v-if="resourceClaimRef"><router-link :to="'/resourceclaim/' + resourceClaimRef.namespace + '/' + resourceClaimRef.name">ResourceClaim {{resourceClaimRef.namespace}}/{{resourceClaimRef.name}}</router-link></p>
  <p><router-link :to="'/resourcepool/createfrom/handle/' + $route.params.namespace + '/' + $route.params.name">Create Pool from Handle</router-link></p>
  <YamlBlob :obj='resourcehandle'/>
</template>

<script>
import YamlBlob from '@/components/YamlBlob.vue'

export default {
  name: 'ResourceHandle',
  components: {
    YamlBlob
  },
  computed: {
    resourceClaimRef () {
      if (this.resourcehandle && this.resourcehandle.spec.resourceClaim) {
        return this.resourcehandle.spec.resourceClaim
      } else {
        return null
      }
    },
    resourcePoolRef () {
      if (this.resourcehandle && this.resourcehandle.spec.resourcePool) {
        return this.resourcehandle.spec.resourcePool
      } else {
        return null
      }
    }
  },
  data () {
    return {
      error: '',
      resourcehandle: ''
    }
  },
  created () {
    window.apiSession
    .then(session => {
      return fetch('/apis/poolboy.gpte.redhat.com/v1/namespaces/' + this.$route.params.namespace + '/resourcehandles/' + this.$route.params.name, {
        headers: {
          'Authentication': 'Bearer ' + session.token
        }
      }); 
    })
    .then(response => {
      if (response.status === 200) {
        response.json().then(data => {
          this.resourcehandle = data
        })
      } else if(response.status === 401) {
        this.error = 'Session expired, please refresh.'
      } else if(response.status === 403) {
        this.error = 'Sorry, it seems you do not have access.'
      } else {
        this.error = response.status
      }
    })
    .catch(error => {
      this.error = error
    })
  },
}
</script>
