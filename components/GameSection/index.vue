<template>

  <v-container md>
    <v-row justify="center">
      <v-col cols="12" sm="8" md="3">

        <game-filter :searchParams="searchParams" :orderBy="orderBy"/>

      </v-col>
      <v-col cols="12" sm="8" md="9">

        <game-list v-if="filteredGames.length" :gameItems="filteredGames" />
        <v-row v-else>
          <v-col cols="12">
            <v-skeleton-loader
              class="mx-auto"
              type="card,list-item-two-line"
            ></v-skeleton-loader>
          </v-col>
        </v-row>

      </v-col>
    </v-row>
  </v-container>


</template>


<script lang="ts">

import {Component, Vue} from 'nuxt-property-decorator'
import GameList from "~/components/GameSection/GameList.vue";
import {Game} from "~/types/game";
import {OrderBy} from "~/types/OrderBy";
import {SearchParams} from "~/types/SearchParams";


@Component({
  components: {GameList}
})

export default class GameSection extends Vue {


  public searchParams:SearchParams = {
    name: '',
    score: 0
  }

  public orderBy:OrderBy = {
    title: 'Score',
    order: 'desc'
  }


  get gameList():{games:Game[], name:String} {
    return (this.$store.state).games
  }


    get filteredGames()
    {
      let games:Game[] =  this.filterByName(this.filterByScore(this.gameList.games))

      if (this.orderBy.title === 'Name')
        return this.sortDataBykey(games, this.orderBy.order, 'name')
      else if (this.orderBy.title === 'Score')
        return this.sortDataBykey(games, this.orderBy.order, 'rating')
      else if (this.orderBy.title === 'Release date')
        return this.sortDataBykey(games, this.orderBy.order, 'first_release_date')
      else
        return games
    }



    public filterByName(gameItems:Game[]):Game[]{
      if (this.searchParams.name)
        return gameItems.filter((game:Game) =>
          ((this.searchParams.name??'').toLowerCase().split(' ').every(item => (game.name??'').toLowerCase().includes(item))))
       else
        return gameItems;
    }

  public filterByScore(gameItems:Game[]):Game[]{
    let score = 0
    if (this.searchParams.score){
      score = this.searchParams.score??0
      return gameItems.filter((game:Game) =>
        ((game.rating??100)/10 >= score))
    }
    else
      return gameItems;
  }


  public sortDataBykey(items:Game[], sortDir:String, sortKey:keyof Game){
    let currentSortDir = sortDir === 'asc' ? 'desc' : 'asc';
    let modifier = -1;
    if(currentSortDir === 'desc') modifier = 1;

    let gamesCopy:Game[] = [...items]
    let AObject:Object = {}
    let BObject:Object = {}
    gamesCopy.sort((a,b) => {
      AObject = a[sortKey]??{}
      BObject = b[sortKey]??{}
      if(AObject > BObject){return modifier}
      if(AObject < BObject){return -1 * modifier}
      return 0
    })
    return gamesCopy
  }



  mounted() {
    this.$store.dispatch('games/fetchGames')
  }


}

</script>




<style>

.games-list .card {
  margin: 20px;
}

</style>
