<script>
import $ from 'jquery'

export default {
    data() {
        return {
            data: {
                case_number: '',
                date: '',
                time: '',
                code: '',
                incident: '',
                police_grid: '',
                neighborhood_number: '',
                block: ''
            },
        }
    },
    methods: {
        caseChange: (comp) => {
            comp.data.case_number = document.getElementById('caseInput').value;
        },
        dateChange: (comp) => {
            comp.data.date = document.getElementById('dateInput').value;
        },
        timeChange: (comp) => {
            comp.data.time = document.getElementById('timeInput').value;
        },
        codeChange: (comp) => {
            comp.data.code = document.getElementById('codeInput').value;
        },
        incidentChange: (comp) => {
            comp.data.incident = document.getElementById('incidentInput').value;
        },
        policeChange: (comp) => {
            comp.data.police_grid = document.getElementById('policeInput').value;
        },
        neighborhoodChange: (comp) => {
            comp.data.neighborhood_number = document.getElementById('neighborhoodInput').value;
        },
        blockChange: (comp) => {
            comp.data.block = document.getElementById('blockInput').value;
        },
        addIncident(){
            let submitForm = true;
            for (let value in this.data){
                if (this.data[value] == null || this.data[value] == ""){
                    submitForm=false;
                }
            }
            if (submitForm == true){
                console.log(this.data.incident);
                let json_data = JSON.stringify({ "case_number": parseInt(this.data.case_number), "date": this.data.date, "time":this.data.time, "code": parseInt(this.data.code), "incident":this.data.incident,"police_grid":parseInt(this.data.police_grid), "neighborhood_number":parseInt(this.data.neighborhood_number), "block":this.data.block});
                console.log(json_data);
                $.ajax({
                    type: 'PUT',
                    contentType: 'application/json',
                    url: 'http://localhost:8080/new-incident',
                    data: json_data,
                    success: (response) => {
                        alert("Successfully added incident to the Database!");
                    },
                    error: (status, message) => {
                        alert("Unable to add incident to the database!");
                    }
                });
            } else {
                alert("Please fill all fields before submitting!");
            }
        }

    },
    mounted() {
    }
}
</script>

<template>
    <form @submit.prevent="addIncident">
        <div class="grid-container">
            <div class="grid-y grid-padding-y">

                <!-- Date/Time/Limit Filters -->
                <div class="grid-x grid-padding-x">
                    <div class="cell small-4">
                        <h5><b>Case Information</b></h5>
                    </div>
                    <div class="cell small-4">
                        <h5><b>Date</b></h5>

                    </div>
                    <div class="cell small-4">
                        <h5><b>Incident Information</b></h5>

                    </div>
                </div>
                <div class="grid-x grid-padding-x">
                    <div class="cell small-4">
                        <div class="input-group">
                            <span class="input-group-label">Case Number</span>
                            <input id="caseInput" class="input-group-field" type="case" @change="caseChange(this)" required/>
                        </div>
                        <div class="input-group">
                            <span class="input-group-label">Police Grid</span>
                            <input id="policeInput" class="input-group-field" type="police_grid" @change="policeChange(this)" required/>
                        </div>
                        <div class="input-group">
                            <span class="input-group-label">Block</span>
                            <input id="blockInput" class="input-group-field" type="block" @change="blockChange(this)" required/>
                        </div>
                    </div>
                    <div class="cell small-4">
                        <div class="input-group">
                            <span class="input-group-label">Date</span>
                            <input id="dateInput" class="input-group-field" type="date" @change="dateChange(this)" required/>
                        </div>
                        <div class="input-group">
                            <span class="input-group-label">Time</span>
                            <input id="timeInput" class="input-group-field" type="time" @change="timeChange(this)" required/>
                        </div>
                    </div>
                    <div class="cell small-4">
                        <div class="input-group">
                            <span class="input-group-label">Incident</span>
                            <input id="incidentInput" class="input-group-field" type="incident" @change="incidentChange(this)" required/>
                        </div>
                        <div class="input-group">
                            <span class="input-group-label">Code Number</span>
                            <input id="codeInput" class="input-group-field" type="code" @change="codeChange(this)" required/>
                        </div>
                        <div class="input-group">
                            <span class="input-group-label">Neighborhood Number</span>
                            <input id="neighborhoodInput" class="input-group-field" type="neighborhood_number" @change="neighborhoodChange(this)" required/>
                        </div>
                    </div>
                    <p class="cell selected" type="submit" @click="addIncident">Add Incident</p>
                </div>
            </div>
        </div>
    </form>
</template>
