<script>
//npm run dev -- --port 8000
import $ from 'jquery'

import About from './About.vue'
import FilterForm from './FilterForm.vue'
import NewIncidentForm from './NewIncidentForm.vue'


export default {
    components: {
        About,
        FilterForm,
        NewIncidentForm
    },
    data() {
        return {
            view: 'map',
            codes: [],
            neighborhoods: [],
            incidents: [],
            filter: false,
            leaflet: {
                map: null,
                center: {
                    lat: 44.955139,
                    lng: -93.102222,
                    address: ""
                },
                zoom: 12,
                bounds: {
                    nw: { lat: 45.008206, lng: -93.217977 },
                    se: { lat: 44.883658, lng: -92.993787 }
                },
                neighborhood_markers: [
                    { location: [44.942068, -93.020521], marker: 'Southeast' },
                    { location: [44.977413, -93.025156], marker: 'Greater East Side' },
                    { location: [44.931244, -93.079578], marker: 'West Side' },
                    { location: [44.956192, -93.060189], marker: "Dayton's Bluff" },
                    { location: [44.978883, -93.068163], marker: 'Payne - Phalen' },
                    { location: [44.975766, -93.113887], marker: 'North End' },
                    { location: [44.959639, -93.121271], marker: 'Frogtown' },
                    { location: [44.947700, -93.128505], marker: 'Summit - University' },
                    { location: [44.930276, -93.119911], marker: 'West Seventh - Fort Road' },
                    { location: [44.982752, -93.147910], marker: 'Como Park' },
                    { location: [44.963631, -93.167548], marker: 'Hamline - Midway' },
                    { location: [44.973971, -93.197965], marker: 'Saint Anthony Park' },
                    { location: [44.949043, -93.178261], marker: 'Union Park' },
                    { location: [44.934848, -93.176736], marker: 'Macalestar - Groveland' },
                    { location: [44.913106, -93.170779], marker: 'Highland' },
                    { location: [44.937705, -93.136997], marker: 'Summit Hill' },
                    { location: [44.949203, -93.093739], marker: 'Downtown' }
                ]
            }
        };
    },
    methods: {
        viewMap(event) {
            this.view = 'map';
        },

        viewNewIncident(event) {
            this.view = 'new_incident';
        },

        viewAbout(event) {
            this.view = 'about';
        },

        goToLocation(event) {
            let el = document.getElementById('location');
            if (!el.value) return;

            if (el.value.split(',').length == 2) {
                this.leaflet.map.flyTo(el.value.split(','), 18);
            } else {
                let loc = el.value.split(" ");
                loc[0].replace('X', '0');
                loc.push("MN");
                loc = loc.join(" ");
                this.getText(`https://nominatim.openstreetmap.org/search?q=${loc}&format=jsonv2`)
                    .then(res => {
                        const parsed = JSON.parse(res);
                        if (parsed.length < 0) return;
                        const location = parsed[0];
                        this.leaflet.map.flyTo([location.lat, location.lon], 18);
                    })
                    .catch(err => {
                        console.log(err);
                    });
            }
        },

        toggleFilter(event) {
            this.filter = !this.filter;
        },

        updateLocation(event, location) {
            let el = document.getElementById('location');
            location = `${location.lat}, ${location.lng}`;
            el.value = location ?? el.value;
            el.dispatchEvent(new Event('input'));
        },

        onMapClick(e) {
            this.updateLocation(null, e.latlng);
        },

        onMapMove(e) {
            this.updateLocation(null, this.leaflet.map.getCenter());
        },

        updateTable(filter) {
            var url = 'http://localhost:8080/incidents?';

            if (filter.incidents.length > 0) {
                url += 'code=' + filter.incidents.join(',') + '&';
            }

            if (filter.neighborhoods.length > 0) {
                url += 'neighborhood=' + filter.neighborhoods.join(',') + '&';
            }

            if (filter.startDate != '') {
                url += 'start_date=' + filter.startDate + '&';
            }

            if (filter.endDate != '') {
                url += 'end_date=' + filter.endDate + '&';
            }

            url += 'limit=' + filter.limit;

            this.getJSON(url).then((result) => {
                if (filter.startTime != '') {
                    result = result.filter(i => i.time > filter.startTime);
                }

                if (filter.endTime != '') {
                    result = result.filter(i => i.time < filter.endTime);
                }

                this.incidents = result;
                this.filter = false;
            }).catch((error) => {
                console.log(error);
            });
        },

        getText(url) {
            return new Promise((resolve, reject) => {
                $.ajax({
                    dataType: 'text',
                    url: url,
                    success: (response) => {
                        resolve(response);
                    },
                    error: (status, message) => {
                        reject({ status: status.status, message: status.statusText });
                    }
                });
            })
        },

        getJSON(url) {
            return new Promise((resolve, reject) => {
                $.ajax({
                    dataType: 'json',
                    url: url,
                    success: (response) => {
                        resolve(response);
                    },
                    error: (status, message) => {
                        reject({ status: status.status, message: status.statusText });
                    }
                });
            });
        },

        uploadJSON(method, url, data) {
            return new Promise((resolve, reject) => {
                $.ajax({
                    type: method,
                    url: url,
                    contentType: 'application/json; charset=utf-8',
                    data: JSON.stringify(data),
                    dataType: 'text',
                    success: (response) => {
                        resolve(response);
                    },
                    error: (status, message) => {
                        reject({ status: status.status, message: status.statusText });
                    }
                });
            });
        }
    },
    mounted() {
        this.leaflet.map = L.map('leafletmap').setView([this.leaflet.center.lat, this.leaflet.center.lng], this.leaflet.zoom);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
            minZoom: 11,
            maxZoom: 18
        }).addTo(this.leaflet.map);

        for (var i = 0; i < this.leaflet.neighborhood_markers.length; i++) {
            var location = this.leaflet.neighborhood_markers[i].location;
            var name = this.leaflet.neighborhood_markers[i].marker;
            var marker = L.marker(location).addTo(this.leaflet.map).on('click', this.onMapClick);
            marker.bindPopup(`Incidents in the ${name} Neighborhood: {Incident # here}`);
        }

        this.leaflet.map.setMaxBounds([[44.883658, -93.217977], [45.008206, -92.993787]]);
        this.leaflet.map.on('click', this.onMapClick);
        this.leaflet.map.on('moveend', this.onMapMove);
        let district_boundary = new L.geoJson();
        district_boundary.addTo(this.leaflet.map);

        this.getJSON('/data/StPaulDistrictCouncil.geojson').then((result) => {
            // St. Paul GeoJSON
            $(result.features).each((key, value) => {
                district_boundary.addData(value);
            });
        }).catch((error) => {
            console.log('Error:', error);
        });

        this.getJSON('http://localhost:8080/neighborhoods').then((result) => {
            this.neighborhood_dict = result;
        }).catch((error) => {
            console.log(error);
        });

        this.getJSON('http://localhost:8080/incidents?limit=1000').then((result) => {
            this.incidents = result;
        }).catch((error) => {
            console.log(error);
        });
    }
}
</script>

<template>
    <div class="grid-container">
        <div class="grid-x grid-padding-x">
            <label for="location">Enter a location here:</label>
            <input class="cell small-4" id="location" v-model="location"
                placeholder="(Lattitude, Longitude) or Address" />
            <button id="updateLocation" class="cell small-4" type="submit" @click="goToLocation">Go</button>
        </div>
    </div>
    <div class="grid-container">
        <div class="grid-x grid-padding-x">
            <p :class="'cell small-4 ' + ((view === 'map') ? 'selected' : 'unselected')" @click="viewMap">Map</p>
            <p :class="'cell small-4 ' + ((view === 'new_incident') ? 'selected' : 'unselected')"
                @click="viewNewIncident">New Incident</p>
            <p :class="'cell small-4 ' + ((view === 'about') ? 'selected' : 'unselected')" @click="viewAbout">About</p>
        </div>
    </div>
    <div v-show="view === 'map'">
        <div class="grid-container">
            <div class="grid-x grid-padding-x">
                <div id="leafletmap" class="cell auto"></div>
            </div>
        </div>
        <br />
        <div>
            <p id="toggleFilter" :class="'cell ' + ((filter) ? 'selected' : 'unselected')" type="submit"
                @click="toggleFilter">Filter</p>
            <div v-show="filter === true">
                <FilterForm @update-rows="updateTable" />
            </div>
        </div>
        <div>
            <table class="table">
                <thead>
                    <tr>
                        <th>Case Number</th>
                        <th>Date</th>
                        <th>Time</th>
                        <th>Incident</th>
                        <th>Police Grid</th>
                        <th>Neighborhood</th>
                        <th>Block</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="incident in incidents">
                        <td>{{ incident.case_number }}</td>
                        <td>{{ incident.date }}</td>
                        <td>{{ incident.time }}</td>
                        <td>{{ incident.incident }}</td>
                        <td>{{ incident.police_grid }}</td>
                        <td>{{ neighborhood_dict.filter(n => n.ID == incident.neighborhood_number)[0].Name }}</td>
                        <td>{{ incident.block }}</td>
                    </tr>
                </tbody>
            </table>

        </div>
    </div>
    <div v-if="view === 'new_incident'">
        <NewIncidentForm />
    </div>
    <div v-if="view === 'about'">
        <!-- Replace this with your actual about the project content: can be done here or by making a new component -->
        <About />
    </div>
</template>

<style>
#leafletmap {
    height: 500px;
}

.selected {
    background-color: rgb(10, 100, 126);
    color: white;
    border: solid 1px white;
    text-align: center;
    cursor: pointer;
}

.unselected {
    background-color: rgb(200, 200, 200);
    color: black;
    border: solid 1px white;
    text-align: center;
    cursor: pointer;
}
</style>
