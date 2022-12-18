<script>

import $ from 'jquery'

export default {
    data() {
        return {
            filtered: {
                case_number: '',
                date: '',
                time: '',
                code: '',
                incident: '',
                police_grid: '',
                neighborhood_number: '',
                block: '',
            },
        }
    },
    methods: {
        caseChange: (comp) => {
            comp.filtered.case_number = document.getElementById('caseNumberInput').value;
        },
        dateChange: (comp) => {
            comp.filtered.date = document.getElementById('dateInput').value;
        },
        timeChange: (comp) => {
            comp.filtered.time = document.getElementById('timeInput').value;
        },
        codeChange: (comp) => {
            comp.filtered.code = document.getElementById('codeInput').value;
        },
        incidentChange: (comp) => {
            comp.filtered.incident = document.getElementById('incidentInput').value;
        },
        policeGridChange: (comp) => {
            comp.filtered.police_grid = document.getElementById('policeGridInput').value;
        },
        neighborhoodChange: (comp) => {
            comp.filtered.neighborhood_number = document.getElementById('neighborhoodInput').value;
        },
        block: (comp) => {
            comp.filtered.block = document.getElementById('block').value;
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

                <form action="submit">
                <div class="grid-x grid-padding-x">
                    <div class="cell small-4">
                        <h5><b>Case Information</b></h5>
                    </div>
                    <div class="cell small-4">
                        <h5><b>Time</b></h5>

                    </div>
                    <div class="cell small-4">
                        <h5><b>Incident Information</b></h5>
                    </div>
                </div>
                <div class="grid-x grid-padding-x">
                    <div class="cell small-4">
                        <div class="input-group">
                            <label>Case Number
                            <input id="caseNumberInput" class="input-group-field" type="text" @change="caseChange(this)" required/></label>
                        </div>
                        <div class="input-group">
                            <label>Crime Code
                            <input id="codeInput" class="input-group-field" type="text" @change="codeChange(this)" /></label>
                        </div>
                    </div>
                    <div class="cell small-4">
                        <div class="input-group">
                            <label>Date
                            <input id="dateInput" class="input-group-field" type="date" @change="dateChange(this)" /></label>
                        </div>
                        <div class="input-group">
                            <label>Time
                            <input id="timeInput" class="input-group-field" type="time" @change="timeChange(this)" /></label>
                        </div>

                        
                    </div>
                    <div class="cell small-4">
                        <label>Incident Type
                        <input id="limitInput" min="1" type="number" :value="filtered.limit" @change="incidentChange(this)" /></label>
                    </div>
                </div>
                </form>

            </div>
        </div>
    </form>
</template>