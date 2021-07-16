<template>
  <v-container>
    <v-row>
      <v-text-field
        label="commit url"
        flat
        rounded
        background-color="light-blue lighten-5"
        solo
        v-model="URL">
        <template #append>
            <v-btn
                rounded
                color="blue"
                @click="getObjects">
            get
            </v-btn>
        </template>
      </v-text-field>
    </v-row>
     
  </v-container>
</template>

<script>
  export default {
    name: 'HelloWorld',
    data: () => ({
      URL: 'https://speckle.xyz/streams/5dfbeb49c9/commits/22bb89b5b8'
    }),
    methods: {
      async getObjects() {

        // get the stream and commit id from the url
        const url = new URL(this.URL)
        var pathArray = url.pathname.split('/')
        const streamId = pathArray[2]
        const commitId = pathArray[4]

        // create a query for graphql for all children of the commit
        const query = this.buildQuery(streamId, commitId)
        console.log(query)

        // fetch
        var server = url.protocol.concat("//").concat(url.host)
        console.log(server)
        let rawRes = await fetch( new URL( '/graphql', server), {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify( {
            query: query,
            variables: { }
          } )
        } )

        let res = await rawRes.json()
        let obj = res.data.stream.object
        console.log(obj)

      },

      buildQuery( streamId, objectId) {
      return `
        query( $mySelect: [String!], $myQuery: [JSONObject!]) {
          stream( id: "${streamId}" ) {
            object( id: "${objectId}" ) {
              data
              children( 
                objects{
                  data
                }
              }
            }
          }
        }
      `
      }
    }
  }
</script>
