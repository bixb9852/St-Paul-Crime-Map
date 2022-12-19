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
            currFilter: null,
            layerGroup: null,
            visibleMarkers: [],
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
                ],
                icons: {}
            },
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
                let location = el.value.split(',');
                if ((44.883658 <= parseFloat(location[0]) <= 45.008206) && ((-93.217977 <= parseFloat(location[1])) && (parseFloat(location[1]) <= -92.993787))) {
                    this.leaflet.map.flyTo(el.value.split(','), 14);
                } else {
                    alert("Please enter coordinates within the St. Paul city boundaries!");
                }
            } else {
                let loc = el.value.split(" ");
                loc[0] = loc[0].replace(/X/g, '0');
                loc.push("MN");
                loc.push("SAINT PAUL");
                loc = loc.join(" ");
                this.getText(`https://nominatim.openstreetmap.org/search?q=${loc}&format=jsonv2`)
                    .then(res => {
                        const parsed = JSON.parse(res);
                        if (parsed.length < 0) return;
                        const location = parsed[0];
                        if ((44.883658 <= location.lat <= 45.008206) && (-93.217977 <= location.lon) && (location.lon <= -92.993787)) {
                            this.leaflet.map.flyTo([location.lat, location.lon], 14);
                        } else {
                            alert("Please enter an address within the St. Paul city boundaries!");
                        }
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
            this.getViewableNeighborhoods();
        },

        onMapClick(e) {
            this.updateLocation(null, e.latlng);
            let zoomLevel = this.leaflet.map.getZoom();
            if (zoomLevel <= 13) {
                zoomLevel = 13
            }
            this.leaflet.map.setView(e.latlng, zoomLevel);
        },

        onMapMove(e) {
            this.updateLocation(null, this.leaflet.map.getCenter());
        },

        updateTable(filter) {
            if (filter.removed == true && this.currFilter != null) {
                url = this.currFilter;
            } else {
                console.log(filter);
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
            }
            this.getJSON(url).then((result) => {
                if (filter.startTime != '') {
                    result = result.filter(i => i.time > filter.startTime);
                }

                if (filter.endTime != '') {
                    result = result.filter(i => i.time < filter.endTime);
                }

                this.currFilter = url;
                this.incidents = result;
                this.filter = false;
                this.placeMarkers(this.layerGroup);
            }).catch((error) => {
                console.log(error);
            });
        },

        getViewableNeighborhoods() {
            var corners = this.leaflet.map.getBounds();
            var ne = corners.getNorthEast();
            var sw = corners.getSouthWest();
            let inRange = []
            for (var i = 0; i < this.leaflet.neighborhood_markers.length; i++) {
                if ((parseFloat(sw.lat) <= parseFloat(this.leaflet.neighborhood_markers[i].location[0]) <= parseFloat(ne.lat)) && (parseFloat(sw.lng) <= this.leaflet.neighborhood_markers[i].location[1]) && (this.leaflet.neighborhood_markers[i].location[1] <= parseFloat(ne.lng))) {
                    inRange.push(i+1);
                }
            }
            console.log(inRange);
            return inRange;
            // var marker = L.marker([ne.lat,ne.lng],{icon:greenIcon}).addTo(this.leaflet.map)
            // var marker = L.marker([sw.lat,sw.lng],{icon:greenIcon}).addTo(this.leaflet.map)
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
        },

        placeMarkers(layerGroup) {
            this.getJSON('http://localhost:8080/neighborhoods').then((result) => {
                this.neighborhoods = result;
                if (this.incidents.length == 0) {
                    this.getJSON('http://localhost:8080/incidents?limit=1000').then((result) => {
                        this.incidents = result;
                        for (var i = 0; i < this.leaflet.neighborhood_markers.length; i++) {
                            var location = this.leaflet.neighborhood_markers[i].location;
                            var name = this.leaflet.neighborhood_markers[i].marker;
                            var marker = L.marker(location, { title: name }).on('click', this.onMapClick);
                            var incidentNumber = 0;
                            for (var j = 0; j < this.incidents.length; j++) {
                                if (this.incidents[j].neighborhood_number == i + 1) {
                                    incidentNumber++;
                                }
                            }
                            marker.bindPopup(`Incidents in the ${name} Neighborhood: ${incidentNumber}`);
                            layerGroup.addLayer(marker);
                        }
                        layerGroup.addTo(this.leaflet.map);
                    })
                        .catch((error) => {
                            console.log(error);
                        });
                } else {
                    for (var i = 0; i < this.leaflet.neighborhood_markers.length; i++) {
                        var name = this.leaflet.neighborhood_markers[i].marker;
                        var incidentNumber = 0;
                        for (var j = 0; j < this.incidents.length; j++) {
                            if (this.incidents[j].neighborhood_number == i + 1) {
                                incidentNumber++;
                            }
                        }
                        layerGroup.eachLayer(function (layer) {
                            if (layer.options.title === name) {
                                layer.bindPopup(`Incidents in the ${name} Neighborhood: ${incidentNumber}`);
                            }
                        })
                    }
                }
            }).catch((error) => {
                console.log(error);
            });
        },

        getRowColor(incident) {
            let code = incident.code
            let personal = [100, 110, 120, 210, 220, 400, 410, 411, 412, 420, 421, 422, 430, 431, 432, 440, 441, 442, 450, 451, 452, 453, 810, 861, 862, 863];
            let property = [300, 311, 312, 313, 314, 321, 322, 323, 324, 331, 333, 334, 341, 342, 343, 344, 351, 352, 353, 354, 361, 363, 364, 371, 372, 373
                , 374, 500, 510, 511, 513, 515, 516, 520, 521, 523, 525, 526, 530, 531, 533, 535, 536, 540, 541, 543, 545, 546, 550, 551, 553, 555, 556, 560, 561
                , 563, 565, 566, 600, 603, 611, 612, 613, 614, 621, 622, 623, 630, 631, 632, 633, 640, 641, 642, 643, 651, 652, 653, 661, 662, 663, 671, 672, 673
                , 681, 682, 683, 691, 692, 693, 700, 710, 711, 712, 720, 721, 722, 730, 731, 732, 900, 901, 903, 905, 911, 913, 915, 921, 922, 923, 925, 931, 933, 941
                , 942, 951, 961, 971, 972, 975, 981, 982, 1400, 1401, 1410, 1415, 1416, 1420, 1425, 1426, 1430, 1435, 1436];
            if (personal.includes(code)) {
                return 'background:#FB8072';
            } else if (property.includes(code)) {
                return 'background:#FFFFB3'
            } else {
                return "background:#80B1D3";
            }
        },

        deleteIncident(incident) {
            let json_data = JSON.stringify({ "case_number": incident.case_number });
            $.ajax({
                type: 'DELETE',
                contentType: 'application/json',
                url: 'http://localhost:8080/remove-incident',
                data: json_data,
                success: (response) => {
                    alert("Successfully removed incident from the Database!");
                    this.updateTable({
                        incidents: [],
                        neighborhoods: [],
                        startTime: '',
                        endTime: '',
                        startDate: '',
                        endDate: '',
                        limit: 1000,
                        removed: true
                    });
                },
                error: (status, message) => {
                    alert("Unable to remove the incident from the database!");
                }
            });
        },
        toggleMarker(incident) {
            const oldMarker = this.visibleMarkers.filter(d => d.case == incident.case_number)[0];
            if (oldMarker) {
                oldMarker.marker.remove(this.leaflet.map);
                this.visibleMarkers = this.visibleMarkers.filter(d => d != oldMarker);
            } else {
                let loc = incident.block.split(" ");
                loc[0] = loc[0].replaceAll(/X/g, '0');
                loc.push("MN");
                loc.push("SAINT PAUL");
                loc = loc.join("+");
                this.getText(`https://nominatim.openstreetmap.org/search?q=${loc}&format=jsonv2`).then(result => {
                    const parsed = JSON.parse(result);
                    if (parsed.length <= 0) return;
                    const location = parsed[0];
                    const marker = L.marker([location.lat, location.lon], { icon: this.leaflet.icons.green });
                    this.visibleMarkers.push({ case: incident.case_number, marker: marker});
                    let cont = $('<div />');
                    cont.html(`Date: ${incident.date}\r\n Time: ${incident.time}, Incident: ${incident.incident}\n <button id="delBut" onclick="${this.leaflet.map.removeLayer(marker)}"><svg xmlns="http://www.w3.org/2000/svg" width="16"
                                    height="16" fill="currentColor" class="bi bi-trash" viewBox="0 0 16 16">
                                    <path
                                        d="M5.5 5.5A.5.5 0 0 1 6 6v6a.5.5 0 0 1-1 0V6a.5.5 0 0 1 .5-.5zm2.5 0a.5.5 0 0 1 .5.5v6a.5.5 0 0 1-1 0V6a.5.5 0 0 1 .5-.5zm3 .5a.5.5 0 0 0-1 0v6a.5.5 0 0 0 1 0V6z" />
                                    <path fill-rule="evenodd"
                                        d="M14.5 3a1 1 0 0 1-1 1H13v9a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V4h-.5a1 1 0 0 1-1-1V2a1 1 0 0 1 1-1H6a1 1 0 0 1 1-1h2a1 1 0 0 1 1 1h3.5a1 1 0 0 1 1 1v1zM4.118 4 4 4.059V13a1 1 0 0 0 1 1h6a1 1 0 0 0 1-1V4.059L11.882 4H4.118zM2.5 3V2h11v1h-11z" />
                                </svg></button>`);
                    marker.bindPopup(cont[0]);
                    marker.addTo(this.leaflet.map).openPopup();
                    $('#delBut').on('click', () => {
                        this.leaflet.map.removeLayer(marker);
                    });
                }).catch((err) => {
                    console.log(err);
                })
            }
        }
    },
    mounted() {
        this.leaflet.map = L.map('leafletmap').setView([this.leaflet.center.lat, this.leaflet.center.lng], this.leaflet.zoom);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
            minZoom: 11,
            maxZoom: 18
        }).addTo(this.leaflet.map);
        this.layerGroup = L.layerGroup().addTo(this.leaflet.map);
        this.placeMarkers(this.layerGroup);
        this.leaflet.map.setMaxBounds([[44.883658, -93.217977], [45.008206, -92.993787]]);
        this.leaflet.map.on('click', this.onMapClick);
        this.leaflet.map.on('moveend', this.onMapMove);
        let district_boundary = new L.geoJson();
        district_boundary.addTo(this.leaflet.map);
        this.leaflet.icons.green = new L.Icon({
            iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png',
            shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
            iconSize: [25, 41],
            iconAnchor: [12, 41],
            popupAnchor: [1, -34],
            shadowSize: [41, 41]
        });

        this.getJSON('/data/StPaulDistrictCouncil.geojson').then((result) => {
            // St. Paul GeoJSON
            $(result.features).each((key, value) => {
                district_boundary.addData(value);
            });
        }).catch((error) => {
            console.log('Error:', error);
        });

    }
}
</script>

<template>
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
            <label for="location">Enter a location here:</label>
            <input class="cell small-4" id="location" v-model="location"
                placeholder="(Lattitude, Longitude) or Address" />
            <button id="updateLocation" class="cell small-4" type="submit" @click="goToLocation">Go</button>
        </div>
    </div>
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
        <div class='my-legend'>
            <div class='legend-scale'>
                <p class='legend-title'>Crime Categories:</p>
                <ul class='legend-labels'>
                    <li><span style='background:#FB8072;'></span>Violent Crimes</li>
                    <li><span style='background:#FFFFB3;'></span>Property Crimes</li>
                    <li><span style='background:#80B1D3;'></span>Other Crimes</li>
                </ul>
            </div>
        </div>
        <div style="overflow-y: scroll; height:333px;">
            <table class="table">
                <thead style="position: sticky; top: -1px">
                    <tr>
                        <th>Case Number</th>
                        <th>Date</th>
                        <th>Time</th>
                        <th>Incident</th>
                        <th>Police Grid</th>
                        <th>Neighborhood</th>
                        <th>Block</th>
                        <th>Show</th>
                        <th>Delete</th>
                    </tr>
                </thead>
                <tbody>
                    <tr class="color-row" v-for="incident in incidents" :style="getRowColor(incident)">
                        <td>{{ incident.case_number }}</td>
                        <td>{{ incident.date }}</td>
                        <td>{{ incident.time }}</td>
                        <td>{{ incident.incident }}</td>
                        <td>{{ incident.police_grid }}</td>
                        <td>{{ neighborhoods.filter(n => n.ID == incident.neighborhood_number)[0].Name }}</td>
                        <td>{{ incident.block }}</td>
                        <td><button @click="toggleMarker(incident)">Add Marker</button></td>
                        <td><button @click="deleteIncident(incident)"><svg xmlns="http://www.w3.org/2000/svg" width="16"
                                    height="16" fill="currentColor" class="bi bi-trash" viewBox="0 0 16 16">
                                    <path
                                        d="M5.5 5.5A.5.5 0 0 1 6 6v6a.5.5 0 0 1-1 0V6a.5.5 0 0 1 .5-.5zm2.5 0a.5.5 0 0 1 .5.5v6a.5.5 0 0 1-1 0V6a.5.5 0 0 1 .5-.5zm3 .5a.5.5 0 0 0-1 0v6a.5.5 0 0 0 1 0V6z" />
                                    <path fill-rule="evenodd"
                                        d="M14.5 3a1 1 0 0 1-1 1H13v9a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V4h-.5a1 1 0 0 1-1-1V2a1 1 0 0 1 1-1H6a1 1 0 0 1 1-1h2a1 1 0 0 1 1 1h3.5a1 1 0 0 1 1 1v1zM4.118 4 4 4.059V13a1 1 0 0 0 1 1h6a1 1 0 0 0 1-1V4.059L11.882 4H4.118zM2.5 3V2h11v1h-11z" />
                                </svg></button></td>
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

.my-legend {
    display:inline-block;
    margin-left: 10px;
}

.my-legend .legend-title {
    text-align: left;
    display: inline;
    float: left;
    padding-right: 5px;
    margin-bottom: 5px;
    font-weight: bold;
    font-size: 90%;
}

.my-legend .legend-scale ul {
    margin: 0;
    margin-bottom: 5px;
    padding: 0;
    float: left;
    list-style: none;
}

.my-legend .legend-scale ul li {
    display: inline-block;
    font-size: 80%;
    list-style: none;
    margin-left: 0;
    padding-right: 5px;
    line-height: 18px;
    margin-bottom: 2px;
}

.my-legend ul.legend-labels li span {
    display: block;
    float: left;
    height: 16px;
    width: 30px;
    margin-right: 5px;
    margin-left: 0;
    border: 1px solid #999;
}

.my-legend a {
    color: #777;
}
</style>
