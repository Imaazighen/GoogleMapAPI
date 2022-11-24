<template>
    <div>
        <section class="ui two column centered grid" style="position:relative;z-index:1;">
        <div class="column" >
            <form class="ui segment large form" style="background-color: transparent; border: none;">
                <div class="ui message red" v-show="error">{{error}}</div>
                <div class="ui segment"  id="kla" style="border: 2px solid cyan; box-shadow: 1px 1px 25px cyan;background-color: black;">
                    <div class="field">
                    <h1 class="ui two column centered grid"  style="-webkit-text-stroke: 1px; -webkit-text-stroke-color: cyan; color: transparent; padding-bottom: 20px; padding-top: 20px;">
                        TROUVEZ LA POSITION EXACTE D'UNE ADRESSE
                    </h1>
                        <div class="ui right icon input large" :class="{loading:spinner}"> 
                            <input style="border: 1px solid cyan; color: greenyellow; background-color: black;" type="text" 
                            placeholder="Entrez une adresse"
                            v-model="address"
                            id="autocomplete"/>
                            <i class="dot circle link icon" @click="locatorButtonPressed"></i>
                        </div>
                    </div>
                    <button class="ui button" >Placer un marqueur</button>
                </div>
            </form>
        </div>
        </section>
        <section id="map"></section>
    </div>
</template>


<script>

import axios from 'axios'

   export default {

    data() {
        return {
            adress: "",
            error:"",
            spinner: false

        }
    },

    mounted() {
        let autocomplete = new google.maps.places.Autocomplete(
            document.getElementById("autocomplete"),
            {
                bounds: new google.maps.LatLngBounds(
                    new google.maps.LatLng(47.68294, 6.49658)
                )
            });

            autocomplete.addListener("place_changed", () => {
                let place = autocomplete.getPlace();
                console.log(place)
                this.showUserLocationOnTheMap(place.geometry.location.lat(), place.geometry.location.lng())
            });
    },

     methods: {
        locatorButtonPressed() {

            this.spinner = true;

            if(navigator.geolocation) {
               navigator.geolocation.getCurrentPosition(
                position => {
                    this.getAdressFrom(
                        position.coords.latitude, 
                        position.coords.longitude
                    );

                    this.showUserLocationOnTheMap(
                        position.coords.latitude,
                        position.coords.longitude
                    );},

               error => {
                this.error = "Impossible de trouver votre adresse. Veuillez l'écrire manuellement.";
                this.spinner = false;
                // console.log(error.message)
               }
               );
            }

            else {
                this.error = error.message;
                this.spinner = false;
                console.log("Your browser does not support geolocation API");
            }
        },

        getAdressFrom(lat, long){
            axios.get("https://maps.googleapis.com/maps/api/geocode/json?latlng=" 
            + lat 
            + "," 
            + long 
            + "&key=AIzaSyBfRZ3K0R7785Lm5M3p4x78h-ahZ8jRVmQ")

            .then(response => {
                    if(response.data.error_message) {
                        this.error = response.data.error_message;
                        console.log(response.data.error_message);
                    } else {
                        this.address = response.data.results[0].formatted_address
                        // console.log(response.data.results[0].formatted_address);
                    }
                    this.spinner = false;
            })

            .catch(error => {
                this.error = error.message;
                this.spinner = false;
                console.log(error.message);
            });
        },
        showUserLocationOnTheMap(latitude, longitude) {

            let map = new google.maps.Map(document.getElementById("map"), {
            zoom:15,
            center: new google.maps.LatLng(latitude, longitude),
            mapTypeId:  google.maps.MapTypeId.ROADMAP
        });

        new google.maps.Marker({
            position: new google.maps.LatLng(latitude, longitude),
            map:map
        })
        }
     }
   };
</script>


 <style>
    .ui.button,
    .dot.circle.icon {
        background-color: cyan;
        color: black;
    }

    .pac-icon{
        display:none;
    }

    .pac-item{
        padding:10px;
        font-size:16px;
        cursor:pointer;
        color: cyan;
    }

    .pac-item:hover{
        background-color: black;
    }

    .pac-item-query{
        font-size: 16px;
    }

    #map{
        position: absolute;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        background-image: url('nv2.gif');
        background-size:cover;
        background-repeat: no-repeat;
    }

    #kla:not(:hover){
        opacity: 65%;
        transition-duration: 500ms;}
</style>