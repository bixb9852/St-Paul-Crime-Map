<script>
import $ from 'jquery'

export default {
    data() {
        return {
        }
    },
    methods: {
    },
    mounted() {

    }
}
</script>

<template>
        <body>
        <div class="About-us">
            <h1 style = "text-align: center;"> About Us </h1>
            <hr>
                <p><b>Eric Bixby</b></p>
                <p>My name is Eric Bixby</p>
                <p>I am a junior studying Computer Science with a minor in Applied Statistics</p>
                <p>I enjoy traveling, playing sports, and spending time with friends and family</p>
            <hr>
            <p><b>Ani Lamichhane</b></p>
                <p>My name is Ani Lamichhance</p>
                <p>Third year at St. Thomas, studying Computer Science</p>
                <p>I enjoy working with HTML, CSS, and JavaScript.</p>
            <hr>
            <p><b>Seth Zerwas</b></p>
                <p>My name is Seth Zerwas</p>
                <p>I am a junior studying Computer Science.</p>
                <p>I enjoy spending my time outdoors and with family.</p>
            <hr> 
        </div>
        <div class = "Tools"> 
            <h1 style = "text-align: center;"> Tools of the Project </h1>
            <hr>
                <p style = "text-align: left;"><b>VueJS</b></p>
                <p style = "text-align: left;">VueJS is the Javascript framework we used on this project. VueJS helps in developing ineractive web interfaces on a website.</p>
                <p style = "text-align: left;">VueJS has various different ways to apply transitions to HTML elements. Much of this is done through the VueJS virtual DOM
                , which is used by other frameworks like React. The changes are not made to the DOM, but rather a replica that is created through JS data structures
                The changes are not made to the DOM, but rather a replica that is created through JS data structures. Later, these changes are made to the real DOM, 
                and the user will see these changes</p>
                <p style = "text-align: left;">This is done by the HTML elements being added or removed from the DOM.</p>
                
            <hr>
            <p style = "text-align: left;"><b>Nominatim API</b></p>
                <p style = "text-align: left;">Nominatim API is used with the OpenStreetMap dataset.</p>
                <p style = "text-align: left;">It is used to index named, numbered, or some unamed features of the OpenStreetMap dataset.
                It is a search API, and does not provoide its own website interface.</p>
                <p style = "text-align: left;">It features differing endpoints for querying the data, like /search which looks for OSM objects by name,
                or /reverse, which searches the OSM by the location of the place.</p>
            <hr>
        </div>
        <div class="Video">
            <h1 style="text-align:center;">Video Demonstration</h1>
            <div style="text-align: center">
            <iframe width="560" height="315" src="https://www.youtube.com/embed/0iB-UN6BwYI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
            </div>
        </div>
        <div class = "Tools"> 
            <h1 style = "text-align: center;"> Interesting Findings of the Project </h1>
            <hr>
            <ul>
                <li><b>Finding Number 1: University of St. Thomas Crime Data</b></li>
                    <p>There seems to be a trend of lower crime rates in the area of St. Thomas. The number of incidents in that neighborhood is much lower than other areas</p>
                <li><b>Finding Number 2: Murder in the last year</b></li>
                    <p>There have only been 5 recorded homicides/murders in St. Paul in the last year. Two of them took place in the Payne/Phalen neighborhood</p>
                <li><b>Finding Number 3: Crimes Committed</b></li>
                    <p>It seems like the most common crimes, in all areas, tend to be property crimes in St. Paul</p>
                <li><b>Finding Number 4: Crimes in areas</b></li>
                    <p>There seems to be a common trend that in the north and north east sides of the city, crimes are much more common than in the south and south west sides</p>
                <li><b>Finding Number 5: Summit Hill and Como Park</b></li>
                    <p>Summit Hill and Como Park have the lowest amount of incidents. They both sit in the 20s for total incidents in the last year. The next lowest are Southeast and Macalester-Groveland </p>
                <li><b>Finding Number 6: Crimes in Downtown Neighborhood</b></li>
                    <p>The most common crime in Downtown St. Paul seems to be theft, with the majority of property crimes in that area being crimes of theft.</p>
            </ul>
            <hr>
        </div>
</body>
</template>