
<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- Search / Results -->
    <div
      class="
        z-20
        flex
        justify-center
        relative
        bg-hero-pattern bg-cover
        px-4
        pt-8
        pb-32
      "
    >
      <!-- Search Input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>
        <div class="flex">
          <input
            v-model="queryIP"
            class="
              flex-1
              py-3
              px-2
              rounded-tl-md rounded-bl-md
              focus:outline-none
            "
            type="text"
            placeholder="Search for any IP address or leave empty to get your ip info"
          />
          <i
            @click="getIpInfo"
            class="
              fas
              fa-chevron-right
              cursor-pointer
              bg-black
              text-white
              px-4
              flex
              justify-center
              items-center
              rounded-tr-md rounded-br-md
            "
          ></i>
        </div>
      </div>

      <!-- IP Info -->
      <IPInfo v-if="IPInfo" :IPinfo="IPinfo" />
    </div>

    <div id="mapid" class="h-full z-10"></div>
  </div>
</template>

<script>
// @ is an alias to /src
import IPInfo from "../components/IPInfo.vue";
import leaflet from "leaflet";
import { ref, onMounted } from "vue";
import axios from "axios";

export default {
  name: "Home",
  components: {
    IPInfo,
  },
  setup() {
    let mymap;
    const queryIP = ref("");
    const IPinfo = ref(null);

    onMounted(async () => {
      try {
        mymap = leaflet.map("mapid").setView([51.505, -0.09], 13);

        await leaflet
          .tileLayer(
            "https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={yourAccessToken}",
            {
              attribution:
                'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
              maxZoom: 18,
              id: "mapbox/streets-v11",
              tileSize: 512,
              zoomOffset: -1,
              accessToken: "yourAccessToken",
            }
          )
          .addTo(mymap);
      } catch (err) {
        console.error(err);
        alert(err);
      }
    });

    const getIpInfo = async () => {
      try {
        const data = await axios(
          `https://geo.ipify.org/api/v1?apiKey={yourGeoLocationAPI_KEY}&ipAddress=${queryIP.value}`
        );
        const result = await data.data;

        IPinfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng,
        };
        leaflet.marker([IPinfo.value.lat, IPinfo.value.lng]).addTo(mymap);
        mymap.setView([IPinfo.value.lat, IPinfo.value.lng], 13);
      } catch (err) {
        console.error(err);
        alert(err);
      }
    };

    return { queryIP, IPinfo, getIpInfo };
  },
};
</script>
