<template>
    <v-card v-if="data" align='center'>
        <v-img
          :src="data.image_url"
          cover
          class="text-white"
          style="height: 300px; width: 100%; object-fit: cover;"
        > </v-img>
        <v-card-text align='left'>
          <h2>
            {{ data.player_name }}
          </h2>
          <p>Club: {{ data.club_name ? data.club_name : "No club" }}</p>
          <p>Country: {{ data.country }}</p>
          <p>Date of birth: {{ datestring }}</p>
          <p>Position: {{ data.position }}</p>
          <p>Foot: {{ data.foot }}</p>
          <p>Height (cm): {{data.height_in_cm}}</p>
          <p>Market value (€): {{ data.market_value_in_eur ? Number(data.market_value_in_eur).toLocaleString(undefined) : "unknown" }}</p>
          <p>Max Market value (€): {{ Number(data.highest_market_value_in_eur).toLocaleString(undefined) }}</p>
          <p>FIFA Rating: {{ stats ? stats.overall : 'Not Found' }}</p>
          <div v-if="stats !== null && stats.shooting !== null">
            <radar-chart :key="stats._id" :pace="stats.pace" :shooting="stats.shooting" :passing="stats.passing" :dribbling="stats.dribbling" :defending="stats.defending" :physic="stats.physic" />
          </div>
        </v-card-text>
    </v-card>
</template>

<script>
import RadarChart from './RadarChart'
export default {
  name: 'PlayerCard',
  components: {
    RadarChart
  },
  props: {  
    id: Number,
    name: String
  }
  ,
  data() {
    return {
        loading: false,
        post: null,
        error: null,
        data: null,
        datestring: null,
        stats: null,
    }
  },
  methods:{
    async sendMongoRequest(name) {
      const encodedName = encodeURIComponent(name);
      const url = `http://localhost:8081/players_fifa/${encodedName}`;
      try {
        const response = await fetch(url, {
          method: 'GET',
          mode: 'cors',
        });
        const data = await response.json();
        console.log(data.data);
        this.stats = data.data;
      } catch (error) {
        console.log(error);
        return { error };
      }
    },
    fetchData(id) {
      const config = require('../../config.json')
        fetch(
          `${config.backend_url}/player_id/${id}`
        ).then(

           res => {
            if (res.ok) {
              //this.data=res.json()
              return res.json()
            }
            else {
              return {
                    player_name: "ERROR!",
                    country: "Belgium",
                    date_of_birth: "1995-01-30",
                    position: "Attack", 
                    foot: "Right", 
                    height_in_cm: "180", 
                    market_value_in_eur: "50000", 
                    image_url: ""
              }
            }
           }
        ).then(
           data => this.data = data
        ).then(() =>
          {
            const date = new Date(this.data.date_of_birth)
            this.datestring = date.toDateString()
          })
    },
    
  },
  mounted() {
    this.fetchData(this.id);
    this.sendMongoRequest(this.name);
  },
  watch: {
    id(newId) {
      this.fetchData(newId)
    },
    name(newName) {
      this.sendMongoRequest(newName);
    }
  }
}
</script>

<style scoped>
.card {
  background-color: #FFFFFF;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin: 10px;
  width: 300px;
  transition: all 0.3s ease-in-out;
}

.card:hover {
  transform: scale(1.03);
  box-shadow: 0 6px 8px rgba(0, 0, 0, 0.15);
}

.text-white {
  color: #FFFFFF;
}

h2 {
  font-size: 24px;
  font-weight: bold;
  margin-top: 20px;
  text-transform: uppercase;
}

p {
  font-size: 16px;
  margin: 10px 0;
}
.v-card-text {
  padding: 20px;
}

.v-card-text p:first-of-type {
  margin-top: 0;
}

.v-card-text p:last-of-type {
  margin-bottom: 0;
}
</style>