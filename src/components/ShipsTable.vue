<template lang=pug>
	v-container(style={position: 'relative'})

		v-data-table(
			:items='ships'
			:headers='headers'
			:items-per-page=7
			loading-text='Fetching data...'
			style={position: 'relative', top: 0, right: 0})

		v-fade-transition(mode='in-out')
			v-btn.ma-5(
				@click='refetch'
				v-bind:key='btn.state'
				:loading='query.loading'
				:color='btn.color'
				:ripple='false'
				style={bottom: 0, left: 0, position: 'absolute'}
				small)

				v-icon {{btn.icon}}

</template>

<script>
  import gql from "graphql-tag";
  import {debounce} from "lodash";

  const apollo = {
    ships:
      gql`{
					ships {
						id
						name
						successful_landings
						}
				}`
  }
  export default {
    name: 'ShipsTable',
    apollo,
    computed: {
      query() {
        return this.$apollo.queries.ships
      },
    },

    created() {
      this.resetButton = debounce(() => {
        this.btn.color = 'blue darken-2'
        this.btn.icon = 'mdi-reload'
        this.btn.state = 'reset'
      }, 2000)
    },


    methods: {
      async refetch() {
        this.resetButton.cancel()
        this.btn.color = 'blue darken-2'
        this.btn.icon = null
        const {data, errors} = await this.query.refetch()
        if (data) {
          this.btn.color = 'green'
          this.btn.icon = 'mdi-check'
          this.btn.state = 'done'

        } else if (errors) {
          this.btn.color = 'red'
          this.btn.icon = 'mdi-alert-circle-outline'
          this.btn.state = 'error'

        }
      }
    },
    watch: {
      'btn.color': function (newColor) {
        if (newColor !== 'blue darken-2') {
          this.resetButton()
        }
      }
    },
    data: () => ({
      btn: {
        color: 'blue darken-2',
        icon: 'mdi-reload',
        state: 'normal'
      },
      loading: {
        ships: true,
      },
      ships2: [],

      headers: [
        {text: "Name", value: 'name'},
        {text: 'ID', value: 'id'},
        {text: 'Successful landings', value: 'successful_landings'}
      ],
    }),

  }
</script>
