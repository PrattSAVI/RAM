
<script>
    import * as L from 'leaflet';
    import { count } from "./store.js";

    import { createEventDispatcher } from 'svelte';
	const dispatch = createEventDispatcher();
   
    export let geojson;
    export let filters;

    let map_green = "var(--green)";

    //countValue is map object stored in the store.js
    let map;
    count.subscribe(value => {
        map = value;
    });

    let tecs = [
		{value:"STECAccess",name:"Public Access" , source:"https://raw.githubusercontent.com/PrattSAVI/RAM/main/img/iconsSv/STECAccess.svg" ,desc:"Improve direct access to the water and create linkages to other recreational areas, as well as provide increased opportunities for fishing, boating, swimming, hiking, education, or passive recreation."},
		{value:"STECAcquisition",name:"Acquisition", source: "https://raw.githubusercontent.com/PrattSAVI/RAM/main/img/iconsSv/STECAcquisition.svg" ,desc:"Protect ecologically valuable coastal lands throughout the Hudson-Raritan Estuary from future development through land acquisition."},
		{value:"STECEelgrass",name:"Eelgrass Beds" , source:"https://raw.githubusercontent.com/PrattSAVI/RAM/main/img/iconsSv/STECEelgrass.svg" ,desc:"Establish eelgrass beds at several locations in the HRE study area."},
		{value:"STECForests",name:"Coastal and Maritime Forests", source: "https://raw.githubusercontent.com/PrattSAVI/RAM/main/img/iconsSv/STECForests.svg" ,desc:"Create a linkage of forests accessible to avian migrants and dependent plant communities."},
		{value:"STECAquaticHab",name:"Habitat for Fish, Crab, and Lobsters" , source:"https://raw.githubusercontent.com/PrattSAVI/RAM/main/img/iconsSv/STECAquaticHab.svg" ,desc:"Create functionally related habitats in each of the eight regions of the Hudson-Raritan Estuary."},
		{value:"STECIslands",name:"Habitats for Waterbirds", source: "https://raw.githubusercontent.com/PrattSAVI/RAM/main/img/iconsSv/STECIslands.svg" ,desc:"Restore and protect roosting, nesting, and foraging habitat (i.e., inland trees, wetlands, shallow shorlines) for long-legged wading birds."},
		{value:"STECOyster",name:"Oyster Reefs" , source:"https://raw.githubusercontent.com/PrattSAVI/RAM/main/img/iconsSv/STECOyster.svg" ,desc:"Establish sustainable oyster reefs at several locations."},
		{value:"STECSediment",name:"Sediment Contamination", source: "https://raw.githubusercontent.com/PrattSAVI/RAM/main/img/iconsSv/STECSediments.svg" ,desc:"Isolate or remove one or more sediment zone(s) that is contaminated until such time as all HRE sediments are considered uncontaminated based on all related water quality standards, related fishing / shelling bans or fish consumption advisories, and any newly-promulgated sediment quality standards, criteria or protocols"},
		{value:"STECShore",name:"Shorelines and Shallows" , source:"https://raw.githubusercontent.com/PrattSAVI/RAM/main/img/iconsSv/STECShore.svg" ,desc:"Create or restore shorline and shallow sites with a vegetated riparian zone, an inter-tidal zone with a stable slope, and illuminated shallow water."},
		{value:"STECTributary",name:"Tributary Connections" , source:"https://raw.githubusercontent.com/PrattSAVI/RAM/main/img/iconsSv/STECTributary.svg" ,desc:"Reconnect and restore freshwater streams to the estuary to provide a range of quality habitats to aquatic organisms."},
		{value:"STECWater",name:"Enclosed and Confined Water" , source:"https://raw.githubusercontent.com/PrattSAVI/RAM/main/img/iconsSv/STECWater.svg" ,desc:"Improve water quality in all enclosed waterways and tidal creeks within the estuary to match or surpass the quality of their receiving waters."},
		{value:"STECWetland",name:"Wetlands" , source:"https://raw.githubusercontent.com/PrattSAVI/RAM/main/img/iconsSv/STECWetland.svg" ,desc:"Create and restore coastal and freshwater wetlands at a rate exceeding the annual loss or degradation, to produce a net gain in acreage."},
	]

    //Zoom to active polygon and write id to store.
    function activePolygon(e){
        map.setView( [e.target._latlng.lat , e.target._latlng.lng] ,13)
        dispatch('message', {
			active: e.target
		});
    }

        //Place icons in the hovers
    function placeIcons(feature){
        let props = feature.properties;
        let site_tecs = []
        tecs.forEach( function(tec){
            if (props[ "Primary TEC" ] == tec.value ){
                site_tecs.push( tec )
            }
        })
        
        if ( site_tecs.length > 0){
            site_tecs = site_tecs;
        }else{
            //Remove icon holder if there are no TECS
            site_tecs = null;
        }

        let text = `<div>`;
        if (site_tecs){
            site_tecs.forEach( function(tec) {
            text = text + `<img class="hover-logos" alt=${tec.value} style="width:25px" src= ${tec.source} />`
            })
        }

        text = text + `</div>`
        return text;
    }

    function onEachFeature(feature, layer) {

        var popupContent = `
        <span class="Retired-popup-type">
            Completed Project
        </span>
        ${placeIcons(feature)}
        <span class="Retired-popup-text">
            ${feature.properties.SiteName}
        </span>`;
        
        
        
        
        layer.bindPopup(popupContent);
        layer.on({
            click:activePolygon,
            mouseover: e => {layer.openPopup()},
            mouseout: e => {layer.closePopup()}

        })
    };

    //Prepare data to be filtered by geojson object
    function dataFilter(feature) {
        //Site Filters
        let site_remain = true;
        if( filters.site_filters.length === 0){
            site_remain = true;
        }else{
            if (filters.site_filters.includes(feature.properties.db) ) {
            site_remain = true;
            } else {
                site_remain = false;
            }
        }

        //STEC Filters
        
        let tec_remain = false;
        if (filters.tec_filters.length === 0){
            tec_remain=true;
        }else{
            filters.tec_filters.forEach(function(filter){
                if (feature.properties['Primary_TEC'] === filter){
                    tec_remain=true;
                }else{
                    tec_remain=false;
                }
            })
        }
        let remain = site_remain && tec_remain 
        return remain;
    }

    //Create new filtered data -> Dispatch this to place-holder
    let map_data = geojson.filter( dataFilter );

    //Remove existing Elements
    let all_paths = document.querySelectorAll('path')
    all_paths.forEach( function(paths){
        if (paths.id.includes("AllPoints")){
            paths.remove()
        }
    })

    // create a vector circle centered on each point feature's latitude and longitude
    function createCircles (feature, latlng) {
        return L.circleMarker(latlng)
    }

    function getStrokeWeight(d) {
        return String(d) == filters.active ? 3 : 1;
    }

    function getRadius(d) {
        return String(d) == filters.active ? 10 : 3;
    }

    function style(feature) {
        return {
        radius: getRadius(feature.properties.CRPID),
        fillColor: map_green,
        weight: getStrokeWeight(feature.properties.CRPID),
        color:"white",
        opacity: 1,
        fillOpacity: 1
        }
    }

    const layer = L.geoJSON(map_data,{
        pointToLayer: createCircles,
        onEachFeature: onEachFeature,
        style:style
    }).addTo(map);

    layer.eachLayer(function (polygon) {
        polygon._path.id = polygon.feature.properties.CRPID + " AllPoints" ;
    });


</script>


<slot />