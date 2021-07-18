<template>
  <v-container>
    <v-row class="mt-12">
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

    <v-data-table
      v-if="URL && URL.length !== 0"
      :headers="headers"
      :items="flattenedObjects"
      :hide-default-footer="true"
      disable-sort
      class="elevation-1 mt-12">
    </v-data-table>
     
  </v-container>
</template>

<script>
  import flatten from 'flat' 

  export default {
    name: 'HelloWorld',
    data: () => ({
      URL: 'https://speckle.xyz/streams/5dfbeb49c9/commits/22bb89b5b8',
      flattenedObjects: [],
      headers: []
    }),
    methods: {
      async getObjects() {

        // get the stream and commit id from the url
        const url = new URL(this.URL)
        var pathArray = url.pathname.split('/')
        const streamId = pathArray[2]
        const commitId = pathArray[4]

        // create a query for the referenced object of this commit base
        const commitQuery = this.commitQuery(streamId, commitId)

        // fetch
        let commitRawRes = await fetch( new URL( '/graphql', url.origin), {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify( {
            query: commitQuery,
            variables: { }
          } )
        } )

        let commitRes = await commitRawRes.json()
        var objectId = commitRes.data.stream.commit.referencedObject

        // create a query for graphql for all children of the commit
        const objectsQuery = this.objectsQuery(streamId, objectId)

        // fetch
        let objectsRawRes = await fetch( new URL( '/graphql', url.origin), {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify( {
            query: objectsQuery,
            variables: { }
          } )
        } )

        let objectsRes = await objectsRawRes.json()
        let children = objectsRes.data.stream.object.children.objects
        console.log(children)

        // flatten these objects to generate table rows. 
        // This will still yield key value pairs in flattened json
        this.flattenedObjects = children.map(o => flatten(o))

        // get the unique keys of flattened objects for the headers
        // then create headers with text and value
        var uniqueHeaders = new Set()
        this.flattenedObjects.forEach(o => Object.keys(o).forEach(o => uniqueHeaders.add(o)))
        uniqueHeaders.forEach(o => this.headers.push({text: o, value: o, sortable:false}))
      },

      objectsQuery( streamId, objectId) {
      return `
        query {
          stream( id: "${streamId}" ) {
            object( id: "${objectId}" ) {
              children {
                objects {
                  data
                }
              }
            }
          }
        }
      `
      },

      commitQuery( streamId, commitId) {
      return `
        query {
          stream( id: "${streamId}" ) {
            commit( id: "${commitId}" ) {
              referencedObject
            }
          }
        }
      `
      }


    }
  }
</script>
