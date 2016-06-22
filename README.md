# How to make a CartoDB map with buttons to filter layers

See link to map [here](http://acresofdata.com/wp-content/uploads/2016/05/Dairy-map-final3.html)

![Mapimage](/valuechainstatic.png)

There are two main parts to this map:
- The map with buttons to filter different points
- The box overlay with text which changes based on which points you've filtered

The buttons filter points based on their category. You need to have your data table laid out with columns for each of the types of categories. In this case the points represent dairy organisations. Each row in the table is a different organisation. The columns represent the different types of organisations e.g. processor, co-operative etc. If the organisation is a co-operative for example then put a 1 in that column and 0s in the rest. The data needs to be laid out in this way for this method to work.

## Put all of the following inside the head

1. Add this to load CartoDB

    ```
  <link rel="stylesheet" href="http://libs.cartocdn.com/cartodb.js/v3/3.11/themes/css/cartodb.css" />
  <script src="http://libs.cartocdn.com/cartodb.js/v3/3.11/cartodb.js"></script>
    ```
    
2. Now you need to write the css styling. Start with setting the height and width of the map and the size, colour and position of the bottom overlaid band

  ```
  /* Set height and width of map */
    #map {width: 100%; height:100%; background: black;}

  /* Set style of bottomband */
    #bottomband {
      width:100%; 
      height:260px; 
      position:absolute; 
      bottom:0px; 
      right:0px; 
      background:black; 
      opacity: 0.3; 
      z-index:10; 
      color:#F5F5F3; 
    }
  ```  
  
3. Next create the styling for the infobox which is the container for the text at the bottom left

   ```  
  /* Set style of infobox */ 
    #infobox {
      color:white;
      float:left;
      font-size: 13px;
      font-family: Helvetica;
      line-height: normal;
    }
  ```

4. Now you need to style the buttons shown in the top right. 
  
 ```
 /* Set style of buttons */  
  #topbutton {position: absolute; top:10px; right: 5px; width: 200px; background: transparent; z-index:10;}
  #topbutton a { 
      display:inline-block;
      cursor:pointer;
      width: 90%;
      margin-top:6px;
      padding-top:3px;
      padding-bottom:3px;
      padding-left: 6px;
      text-align: left;
      font: bold 11px "Helvetica",Arial;
      line-height: normal;
      color: #555;
      border-radius: 4px;
      border: 1px solid #777777;
      text-decoration: none;
      cursor: pointer;
    }
  #topbutton a.selected,
  #topbutton a:hover { 
     background-color:#CDD2D4;
    }
  ```
5. Then set your font style

 ```
   /* Font styling */
p {
    font-size: 13px;
    font-family: Helvetica;
    color: white;
    font-weight: 400;
   }
h1 {
    font-size: 16px;
    font-family: Helvetica;
    color: white;
    font-weight: 700;
   }
```

6. Now set the colour of each of the map points based on their category. You need to change raw_data_1 for the name of your data table in CartoDB and change the names of each category to the names of your categories.

```
/* Styling of map points */
#raw_data_1 {
   marker-fill-opacity: 0.9;
   marker-line-color: #FFF;
   marker-line-width: 1;
   marker-line-opacity: 1;
   marker-placement: point;
   marker-type: ellipse;
   marker-width: 10;
   marker-allow-overlap: true;
}
#raw_data_1[type="Brookside"] {
   marker-fill: #A6CEE3;
}
#raw_data_1[type="DFCS"] {
   marker-fill: #FF6600;
}
#raw_data_1[type="Githunguri"] {
   marker-fill: #B2DF8A;
}
#raw_data_1[type="KCC"] {
   marker-fill: #33A02C;
}
#raw_data_1[type="Mini-dairy"] {
   marker-fill: #3B007F;
}
#raw_data_1[type="Other processor"] {
   marker-fill: #3E7BB6;
}
#raw_data_1[type="SHG"] {
   marker-fill: #FDBF6F;
}
```

7. Now create a css style for the little circles shown in each of the buttons

```
/* Styling of bullets within buttons */
.bullet {
float: left;
    margin: 2px 10px 5px 2px;
    width: 4px;
    height: 4px;
    border-radius: 50%;
    padding: 2px;
   border: 1px solid rgba(0,0,0,0.2);
    z-index: 1000;
}
```

8. Now you need to build functions to create the map layers. Start by initiating the leaflet map. You can adjust the GPS coordinates of the centre point (currently set to -1,38) and adjust the zoom of the map (currently set to 7).

```
var map;
    function init(){
  // initiate leaflet map
  map = new L.Map('map', { 
    center: [-1,38],
    zoom: 7
  })

  L.tileLayer('http://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png', {
  attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, &copy; <a href="http://cartodb.com/attributions">CartoDB</a>'
}).addTo(map);
```

9. Then insert the link to you cartoDB json

```
  var layerUrl = 'https://georgiabaz.cartodb.com/api/v2/viz/2f7f0d82-104c-11e6-b563-0e98b61680bf/viz.json';
```

10. Then use this code to create the layers. Make sure you change raw_data_1 for the name of your data table and IsCoop, IsSHG etc. for the names of your columns.

```
  var sublayers = [];

  cartodb.createLayer(map, layerUrl)
  .addTo(map)
  .on('done', function(layer) {
    // change the query for the first layer, change raw_data_1 for the name of your data table
    var subLayerOptions = {
      sql: "SELECT * FROM raw_data_1",
    }

    var sublayer = layer.getSubLayer(0);

    sublayer.set(subLayerOptions);

    sublayers.push(sublayer);
  }).on('error', function() {
    //log the error
  });

  //we define the queries that will be performed when we click on the buttons, by modifying the SQL of our layer, raw_data_1 is the name of the data table
  var LayerActions = {
    DFCS: function(){
    sublayers[0].set({
      sql: "SELECT * FROM raw_data_1 WHERE IsCoop = 1",
      });
      return true;
    },
   SHG: function(){
    sublayers[0].set({
      sql: "SELECT * FROM raw_data_1 WHERE IsSHG = 1",
      });
      return true;
    },
    KCC: function(){
    sublayers[0].set({
      sql: "SELECT * FROM raw_data_1 WHERE IsKCC = 1",
      });
      return true;
    },
    Brookside: function(){
    sublayers[0].set({
      sql: "SELECT * FROM raw_data_1 WHERE IsBrookside = 1",
      });
      return true;
    },
    Githunguri: function(){
    sublayers[0].set({
      sql: "SELECT * FROM raw_data_1 WHERE IsGithunguri = 1",
      });
      return true;
    },
    OthPro: function(){
    sublayers[0].set({
      sql: "SELECT * FROM raw_data_1 WHERE IsOthPro = 1",
      });
      return true;
    },
    MDCI: function(){
    sublayers[0].set({
      sql: "SELECT * FROM raw_data_1 WHERE IsMDCI = 1",
      });
      return true;
    },
    all: function() {
      sublayers[0].set({
        sql: "SELECT * FROM raw_data_1",
      });
      return true;
    }
  }

  $('.button').click(function() {
    $('.button').removeClass('selected');
    $(this).addClass('selected');
    //this gets the id of the different buttons and calls to LayerActions which responds according to the selected id
    LayerActions[$(this).attr('id')]();
  });
}  
```

11. Finally create a function that replaces the content of the infowindow with the content held in the button code

```
 <script>
function ReplaceContentInContainer(id,content) {
var container = document.getElementById(id);
container.innerHTML = content;
}
</script>
```

## Put all of the following inside the body

1. Load the map and create a container for the map

```
<body onload="init()">

  <div id='map'></div>
```

2. Now add the map buttons, replace the text about dairy with whatever you want to see in the infowindow when each button is clicked. Replace the text before </a> with the text you want in each button.

```
 <div id="topbutton">
      <a href="#all" id="all" class="button all" 
      onclick="ReplaceContentInContainer('infobox',
      ' <h1>THE DAIRY VALUE CHAIN - KENYA</h1>Dairy is the largest agricultural sub-sector in Kenya, at ~4% of GDP and 19% of AgGDP. Approximately 5 billion litres of milk are produced each year with the vast majority (over 80%) produced by small-holder farmers. Around 55% of the milk produced each year is marketed with the rest used for home consumption. Production is growing at approximately 3% per year due to increased organisation in the value chain and growing consumer demand.<h1>HOW TO USE THIS MAP</h1>The diagram to the right shows the flow of milk (in litres) through various actors in the value chain. Click on the buttons on the top right to see these actors on the map.'
      )">See all points</a>

      <a href="#DFCS" id="DFCS" class="button DFCS" 
      onclick="ReplaceContentInContainer('infobox',
        '<h1>DFCSs (DAIRY FARMER CO-OPERATIVE SOCIETIES)</h1> DFCSs (Dairy Farmer Co-operative Societies) are registered membership organisations of small-scale dairy farmers. One of the core functions of DFCSs in Kenya is the marketing of members produce. DFCSs also typically facilitate production of milk through training, input supply, provision of financial services. Larger DFCSs often also provide facilities for collecting and chilling milk. a<p>Dairy experts estimate that about <b>60% of small scale dairy farmers are members of DFCSs and over 76% of dairy produce is marketed through co- operatives.</b></p>Value chain reports suggest there are between 80-120 DFCSs in Kenya. <b>Data mining across various online sources has identified 105 DFCSs</b> which are shown on this map. The map shows that DFCSs are clustered around the main milk production regions in Central and Western Kenya'
        )">
        <div class="bullet" style="background: #FF6600"></div>
        DFCSs (Dairy Farmer Co-operative Societies)</a>

    <a href="#SHG" id="SHG" class="button SHG" 
      onclick="ReplaceContentInContainer('infobox',
        '<h1>DFCSs (DAIRY FARMER SELF-HELP GROUPS)</h1>DFSHGs are similar to DFCSs but are typically more informal as they do not require official registration. They typically support farmers with marketing, collection of milk and farming advice.<p>Value chain reports suggest there are around 200 DFSHGs in Kenya. <b>Data mining across various online sources has identified 23 DFSHGs.</b> DFSHGs are more difficult to identify and map through online sources as they small, informal organisations.'
        )">
        <div class="bullet" style="background: #FDBF6F"></div>
        DFSHGs (Dairy Farmer Self-Help Groups)</a>

    <a href="#Brookside" id="Brookside" class="button Brookside" 
      onclick="ReplaceContentInContainer('infobox',
         '<h1>BROOKSIDE</h1><b>Brookside Dairies is the largest milk processing company in Kenya</b>, controling 38% of the dairy market in 2015.<p>Brookside produces a variety of dairy products including fresh and powdered milk, yoghurt and butter and distributes across <b>Kenya, Tanzania and Uganda.</b></p><p>The Brookside HQ is located in Ruiru with a network of <b>40 collection and cooling centres across Central and Western Kenya</b> (as seen on the map). The centres source milk from DFCSs, DFSHGs as well as some individual farmers. Brookside provides a number of incentives to active suppliers such as farmer training, links to input suppliers and milk collection'
         )">
         <div class="bullet" style="background: #A6CEE3"></div>
         Brookside</a>

    <a href="#KCC" id="KCC" class="button KCC" 
       onclick="ReplaceContentInContainer('infobox',
       '<h1>NEW KCC</h1>State-owned <b>New Kenya Co-operative Creameries</b> is the third largest processor of milk in Kenya, controlling approximately 15% of the raw milk market in 2015. KCC produces a variety of products primarily under the KCC and Gold Crown brands.<p>The New KCC HQ is located in Nairobi with a network of <b>30 collection and cooling centres across Central and Western Kenya</b> (as seen on the map). The centres source milk from DFCSs, DFSHGs as well as some individual farmers. KCC provides a number of incentives to active suppliers such as farmer training, artificial insemination of cattle and milk collection</p><p>New KCC market share has almost halved in the last 5 years and recent reports suggest plans to privatise the organisation are at an advanced stage'
       )">
       <div class="bullet" style="background: #33A02C"></div>
       KCC</a>

    <a href="#Githunguri" id="Githunguri" class="button Githunguri" 
      onclick="ReplaceContentInContainer('infobox',
      '<h1>GITHUNGURI</h1><b>Githunguri</b> is the second largest processor of milk in Kenya, controlling approximately 16% of the raw milk market in 2015. Githunguri is a <b>farmers cooperative society</b> where producers are the owners of the processing plant and are 100% involved in the decision-making process. The raw milk is supplied by farmers in the Githunguri DFCS.<p>Githunguri products <b>trade under the name FRESHA</b> and the range of products includes Fresha Fresh, Fresha Mala and Fresha Yoghurt.</p><p>Githunguri HQ and processor is located in Githunguri Town, Kiambu and provides services such as feeds, AI, credits, transport, annual bonus, training and extension amongst others'
      )">
      <div class="bullet" style="background: #B2DF8A"></div>
      Githunguri</a>

    <a href="#OthPro" id="OthPro" class="button OthPro" 
      onclick="ReplaceContentInContainer('infobox',
      '<h1>OTHER PROCESSORS</h1>The market share of other processors is <b>estimated at around 31% of the total raw milk intake</b>. In 2011 the two largest of these other processors were SAAL Daima and Buzeki Dairy but since then they have both been acquired by Brookside.<p>The other processors are concentrated around the Nairobi area, Nakuru and Western Kenya. These processors typically source milk from local DFCSs and SHGs as well as some individual farmers. Some also provide services such as collection, training and credit to farmers.'  
      )">
      <div class="bullet" style="background: #3E7BB6"></div>
      Small processors</a>

    <a href="#MDCI" id="MDCI" class="button MDCI" 
      onclick="ReplaceContentInContainer('infobox',
      '<h1>MINI-DAIRIES AND COTTAGE INDUSTRIES</h1>Recent reports suggests there are around <b>140 mini-dairies and cottage industries</b> in Kenya. These organisations typically produce, process and market their own milk, selling to local consumers. Some are specialised focusing on a single product such as cheese or ice cream.<p>86 of these organisations were able to be located and mapped showing that they are <b>distributed widely across Central and Western Kenya as well as the Coastal region</b></p>'
      )">
      <div class="bullet" style="background: #3B007F"></div>
      Mini-dairies & cottage industries</a>    
      </div>
  ```

3. Create a container for the bottom grey band and the infowindow container. Change the text about dairy to whatever you want the infowindow to say before any buttons are clicked.

```
 <div id="bottomband"></div>

  <div style="width:50%; height:260px; position:absolute; bottom:0px; left:0px; z-index:20;">
     <div style="width:90%; height: 230px; margin:10 auto;">
     <div id="infobox">
     <h1>THE DAIRY VALUE CHAIN - KENYA</h1>
     Dairy is the largest agricultural sub-sector in Kenya, at ~4% of GDP and 19% of AgGDP.
     Approximately 5 billion litres of milk are produced each year with the vast majority (over 80%) produced by small-holder farmers. Around 55% of the milk produced each year is marketed with the rest used for home consumption. Production is growing at approximately 3% per year due to increased organisation in the value chain and growing consumer demand.<h1>HOW TO USE THIS MAP</h1>The diagram to the right shows the flow of milk (in litres) through various actors in the value chain. Click on the buttons on the top right to see these actors on the map.
     </div>
     </div>
  </div>
```

4. Optional: Add an image at the bottom right like this. In this case I've added a Sankey diagram to show the flow of dairy produce through the value chain.

```
<div style="width:45%; height:230px; position:absolute; bottom:20px; right:5%; z-index:20;">
           <img src="http://acresofdata.com/wp-content/uploads/2016/05/valuechainimagefinal.png" width="100%" height="100%"/>
  </div>
```

That's it!
