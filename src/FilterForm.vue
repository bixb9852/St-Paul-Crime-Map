<script>
import $ from 'jquery'

export default {
    data() {
        return {
            filtered: {
                incidents: [],
                neighborhoods: [],
                startTime: '',
                endTime: '',
                startDate: '',
                endDate: '',
                limit: 1000,
                removed: false
            },
            incident_names: {
                'Murder': [100, 110, 120],
                'Rape': [210, 220],
                'Robbery': [300, 311, 312, 313, 314, 321, 322, 323, 324, 331, 333, 334, 341, 342, 343, 344, 351, 352, 353, 354, 361, 363, 364, 371, 372, 373, 374],
                'Aggravated Assault': [400, 410, 411, 412, 420, 421, 422, 430, 431, 432, 440, 441, 442, 450, 451, 452, 453],
                'Burglary': [500, 510, 511, 513, 515, 516, 520, 521, 523, 525, 526, 530, 531, 533, 535, 536, 540, 541, 543, 545, 546, 550, 551, 553, 555, 556, 560, 561, 563, 565, 566],
                'Theft': [600, 603, 611, 612, 613, 614, 621, 622, 623, 630, 631, 632, 633, 640, 641, 642, 643, 651, 652,653, 661, 662, 663,671, 672, 673, 681, 682, 683, 691,692, 693],
                'Motor Vehicle Theft': [700, 710, 711, 712, 720,721, 722, 730,731, 732],
                'Assault': [810, 861, 862, 863],
                'Arson': [900, 901, 903, 905, 911, 913, 915, 921, 922, 923, 925, 931, 933, 941, 942, 951, 961, 971, 972, 975, 981, 982],
                'Criminal Damage to Property': [1400, 1401, 1410, 1415, 1416, 1420, 1425, 1426, 1430, 1435, 1436],
                'Narcotics': [1800, 1810, 1811,1812, 1813, 1814, 1815, 1820, 1822, 1823, 1824, 1825, 1830, 1835, 1840, 1841, 1842, 1843, 1844, 1845, 1850, 1855, 1860,  1865,  1870,  1880, 1885],
                'Weapons': [2619],
                'Death Investigation': [3100],
                'Proactive Police Visit': [9954],
                'Community Engagment Event': [9959],
                'Proactive Foot Patrol': [9986]
            },
            neighborhood_names: {
                'Conway/Battlecreek/Highwood': 1,
                'Greater East Side': 2,
                'West Side': 3,
                'Dayton\'s Bluff': 4,
                'Payne/Phalen': 5,
                'North End': 6,
                'Thomas/Dale(Frogtown)': 7,
                'Summit/University': 8,
                'West Seventh': 9,
                'Como': 10,
                'Hamline/Midway': 11,
                'St. Anthony': 12,
                'Union Park': 13,
                'Macalester-Groveland': 14,
                'Highland': 15,
                'Summit Hill': 16,
                'Capitol River': 17
            }
        }
    },
    methods: {
        incidentChange: (e, comp) => {
            for (var i = 0; i < e.length; i++) {
                if (comp.filtered.incidents.indexOf(e[i]) != -1) {
                    comp.filtered.incidents = comp.filtered.incidents.filter(inc => inc != e[i]);
                } else {
                    comp.filtered.incidents.push(e[i]);
                }
            }
        },
        neighborhoodChange: (e, comp) => {
            if (comp.filtered.neighborhoods.indexOf(e) != -1) {
                comp.filtered.neighborhoods = comp.filtered.neighborhoods.filter(n => n != e);
            } else {
                comp.filtered.neighborhoods.push(e);
            }
        },
        limitChange: (comp) => {
            comp.filtered.limit = document.getElementById('limitInput').value;
        },
        startDateChange: (comp) => {
            comp.filtered.startDate = document.getElementById('startDateInput').value;
        },
        endDateChange: (comp) => {
            comp.filtered.endDate = document.getElementById('endDateInput').value;
        },
        startTimeChange: (comp) => {
            comp.filtered.startTime = document.getElementById('startTimeInput').value;
        },
        endTimeChange: (comp) => {
            comp.filtered.endTime = document.getElementById('endTimeInput').value;
        }

    },
    mounted() {

    }
}

</script>

<template>
    <form>
        <div class="grid-container">
            <div class="grid-y grid-padding-y">

                <!-- Date/Time/Limit Filters -->
                <div class="grid-x grid-padding-x">
                    <div class="cell small-4">
                        <h5><b>DATE</b></h5>
                    </div>
                    <div class="cell small-4">
                        <h5><b>TIME</b></h5>

                    </div>
                    <div class="cell small-4">
                        <h5><b>INCIDENT LIMIT</b></h5>

                    </div>
                </div>
                <div class="grid-x grid-padding-x">
                    <div class="cell small-4">
                        <div class="input-group">
                            <span class="input-group-label">Start Date</span>
                            <input id="startDateInput" class="input-group-field" type="date" @change="startDateChange(this)" />
                        </div>
                        <div class="input-group">
                            <span class="input-group-label">End Date</span>
                            <input id="endDateInput" class="input-group-field" type="date" @change="endDateChange(this)" />
                        </div>
                    </div>
                    <div class="cell small-4">
                        <div class="input-group">
                            <span class="input-group-label">Start Time</span>
                            <input id="startTimeInput" class="input-group-field" type="time" @change="startTimeChange(this)" />
                        </div>
                        <div class="input-group">
                            <span class="input-group-label">End Time</span>
                            <input id="endTimeInput" class="input-group-field" type="time" @change="endTimeChange(this)" />
                        </div>
                    </div>
                    <div class="cell small-4">
                        <input id="limitInput" min="1" type="number" :value="filtered.limit" @change="limitChange(this)" />
                    </div>
                </div>

                <!-- Incident Filters -->
                <div class="cell small-2">
                    <h5><b>INCIDENTS</b></h5>
                    <div class="grid-x grid-padding-x">
                        <div class="cell small-4" v-for="(value, key) in incident_names">
                            <input type="checkbox" @change="incidentChange(value, this)" /> {{key}}
                        </div>
                    </div>
                    <h5><b>NEIGHBORHOODS</b></h5>
                    <div class="grid-x grid-padding-x">
                        <div class="cell small-4" v-for="(value, key) in neighborhood_names">
                            <input type="checkbox" @change="neighborhoodChange(value, this)" /> {{key}}
                        </div>
                    </div>
                    <p id="toggleFilter" class="cell selected" type="submit" @click="$emit('updateRows', filtered)">Update</p>
                </div>
            </div>
        </div>
    </form>
</template>