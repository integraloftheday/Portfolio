---
title: Covid Graphs
metaDescription: Covid Graphs
date: 2019-01-01T00:00:00.000Z
author: Mason Manetta
summary: Covid Graphs
tags:
  - Visualization 
---

# ASU Covid Visualizations 

{% Notebook %}

# %% [markdown]
# ASU COVID-19 Plots

An updating plot based on current ASU COVID-19 Data

### Proxy To Get COVID-19 Data
# %%--- [javascript]
# properties:
#   run_on_load: true
# ---%%
var asuCovidUrl = "https://eoss.asu.edu/health/announcements/coronavirus/management/index.html";
var proxyUrl = "https://api.allorigins.win/raw?url=";
var graphWidth = 900;
var graphHeight = 500;


var data = await fetch(proxyUrl+asuCovidUrl);
var htmlData = await data.text();
# %% [markdown]
### Definition of Classes 
# %%--- [javascript]
# properties:
#   run_on_load: true
#   top_hidden: true
# ---%%
class DataEntry{

  constructor(pannelObject){
    this.parser = new DOMParser();
	this.htmlObject = pannelObject; 
    this.date = this.extractDate(); 
    this.currentPositive = this.extractCurrentPositive()
    
  }

  extractDate(){
  	var dateStr = this.htmlObject.getElementsByTagName("p")[0].innerHTML
  		.replace(/&nbsp;/g, '')
  		.replace("Updated: ","")
      	.replace(".","");
  
	dateStr = dateStr.substring(0, dateStr.lastIndexOf('@'));
	//console.log(dateStr);
    return new Date(dateStr)
  }

  extractBetweenH4(index){
    try{
      var extactStr = this.htmlObject.innerHTML.split("<h4>")[index].split("</h4>")[index];
      var title = this.htmlObject.getElementsByTagName("h4")[index].innerHTML;
      var body = this.parser.parseFromString(extactStr,"text/html");
      body = body.getElementsByTagName("body")[0]; 
      return {title: title , body: body};
      }
    catch(e){
      console.log("Parse Error");
      console.log(this.htmlObject);
      console.log(e);
      return null
    }
  }

  extractCurrentPositive(){
    var currentPositive = this.extractBetweenH4(1);
    if(currentPositive != null){
      var intArrayStaff = currentPositive.body.getElementsByTagName("li")[0].innerHTML
      	.replace(/&nbsp;/g, ' ')
      	.replace(",","")
      	.replace("*","")
      	.match(/\\d+/g)
      	.map((x)=>parseInt(x))
  
      var intArrayStudent = currentPositive.body.getElementsByTagName("li")[1].innerHTML
      	.replace(/&nbsp;/g, ' ')
      	.replace(",","")
      	.replace("*","")
      	.match(/\\d+/g)
      	.map((x)=>parseInt(x))
  
      return {
        staffPositive: intArrayStaff.length>=1?intArrayStaff[0] : null,
        staffTotal: intArrayStaff.length>=2?intArrayStaff[1] : null,
        staffPercent: (intArrayStaff.length>=2 & intArrayStaff[1]!=0)?intArrayStaff[0]/intArrayStaff[1] : null,
        studentPositive: intArrayStudent.length>=1?intArrayStudent[0] : null,
        studentTotal: intArrayStudent.length>=2?intArrayStudent[1] : null,
        studentPercent: (intArrayStudent.length>=2 & intArrayStudent[1]!=0)?intArrayStudent[0]/intArrayStudent[1] : null,
      }
  	}
  }
}

class DataFrame{
  constructor(entryArray){
    this.data = {};
    //this.csv ="";

    for(var i = 0; i < entryArray.length; i++){
    	var keys = Object.keys(entryArray[i])
      	var keysExisting = Object.keys(this.data);
      	//console.log(keysExisting);
      	for(var j = 0; j< keys.length; j++){
        //  console.log(this.data);
        	if(keysExisting.includes(keys[j])){
              this.data[keys[j]].push(entryArray[i][keys[j]]);
            } 
            else{
              this.data[keys[j]] = [];
              this.data[keys[j]].push(entryArray[i][keys[j]]);
            }
        }
    }
  }

  delta(dataKey){
    var dataArray = typeof dataKey == 'string' ? this.data[dataKey] : dataKey
    var returnArray = [];
    for(var i = 0; i < dataArray.length -1; i++){
      returnArray.push(dataArray[i+1] - dataArray[i]);
    }
    return returnArray
  }

  div(dataKey1, dataKey2){
    var dataArray1 = typeof dataKey1 == 'string' ? this.data[dataKey1] : dataKey1
    var dataArray2 = typeof dataKey2 == 'string' ? this.data[dataKey2] : dataKey2
    var returnArray = []; 
    //console.log(dataArray1);
    for(var i =0 ; i < dataArray1.length; i++){
      returnArray.push(dataArray1[i]/dataArray2[i])
    }
    return(returnArray);
  }
  
    sub(dataKey1, dataKey2){
    var dataArray1 = typeof dataKey1 == 'string' ? this.data[dataKey1] : dataKey1
    var dataArray2 = typeof dataKey2 == 'string' ? this.data[dataKey2] : dataKey2
    var returnArray = []; 
    for(var i =0 ; i < dataArray1.length; i++){
      returnArray.push(dataArray1[i]-dataArray2[i])
    }
    return(returnArray);
  }
    mult(dataKey1, dataKey2){
    var dataArray1 = typeof dataKey1 == 'string' ? this.data[dataKey1] : dataKey1
    var dataArray2 = typeof dataKey2 == 'string' ? this.data[dataKey2] : dataKey2
    var returnArray = []; 
    for(var i =0 ; i < dataArray1.length; i++){
      returnArray.push(dataArray1[i]*dataArray2[i])
    }
    return(returnArray);
  }

  add(dataKey1, dataKey2){
    var dataArray1 = typeof dataKey1 == 'string' ? this.data[dataKey1] : dataKey1
    var dataArray2 = typeof dataKey2 == 'string' ? this.data[dataKey2] : dataKey2
    var returnArray = []; 
    for(var i =0 ; i < dataArray1.length; i++){
      returnArray.push(dataArray1[i]+dataArray2[i])
    }
    return(returnArray);
  }
  
  get csv(){
    var headerValues = Object.keys(this.data); 
    var headerString = headerValues.join(",") + "\\n"; 
    var bodyString = "";
    for(var i = 0; i < this.data[headerValues[0]].length; i ++){
      var lineStr = ""
      for(var keyI = 0; keyI < headerValues.length ; keyI ++){
        if(keyI == 0)
        	lineStr = this.data[headerValues[keyI]][i].toString()
        else
          	lineStr = lineStr +"," + this.data[headerValues[keyI]][i].toString();
      }
      bodyString = bodyString + "\\n" + lineStr;
    }

    return (headerString + "\\n" + bodyString);
  }
}

# %% [markdown]
### Parse Data
# %%--- [javascript]
# properties:
#   run_on_load: true
# ---%%

var parser = new DOMParser();

// Parse the text
var doc = parser.parseFromString(htmlData, "text/html");

//split data by pannels 
var docPannels = doc.getElementsByClassName("panel"); 

//iterate over 
var docPannelsArray = Object.values(docPannels);

var dataArray = []
for(var i = 0; i<docPannelsArray.length; i++){
	//extract date 
	dataArray.push(new DataEntry(docPannelsArray[i]));
}

dataArray.reverse() // flip data from newest to oldest to oldest to newest

// create proper format for Dataframe
//an array of dictionary entries
var dataInput = [];
dataArray.map((x)=>{if(x.currentPositive != undefined){;
                    
                    	dataInput.push({date:x.date,
                                   staffPositive:x.currentPositive.staffPositive,
                                   staffTotal:x.currentPositive.staffTotal,
                                   staffPercent:x.currentPositive.staffPercent,
                                   studentPositive:x.currentPositive.studentPositive,
                                   studentTotal:x.currentPositive.studentTotal,
                                   studentPercent:x.currentPositive.studentPercent}
                                      )}
                   }
             );

var df = new DataFrame(dataInput)

# %% [markdown]
### Graph Data
# %%--- [javascript]
# properties:
#   run_on_load: true
#   top_hidden: true
# ---%%
await import ("https://cdn.plot.ly/plotly-2.8.3.min.js");

const graphStaff = document.createElement("div")

var trace1 = {
  x: df.data.date,
  y: df.data.staffPositive ,
  name: 'Number of Staff Positive',
  type: 'scatter'
};


var trace2 = {
  x: df.data.date,
  y: df.data.staffPercent.map((x)=>x*100),
  name: 'Percentage of Staff Positive',
  yaxis: 'y2',
  type: 'scatter'
};


var data = [trace1, trace2];

var layout = {
  autosize: true,
  width: graphWidth,
  height: graphHeight,
  title: 'ASU Staff Covid',
  yaxis: {title: 'Number of Staff Positive'},
  yaxis2: {
    title: 'Percentage of Staff Positive',
    titlefont: {color: 'rgb(148, 103, 189)'},
    tickfont: {color: 'rgb(148, 103, 189)'},
    overlaying: 'y',
    side: 'right'
  }
};

Plotly.newPlot(graphStaff, data, layout);

graphStaff
# %%--- [javascript]
# properties:
#   run_on_load: true
#   top_hidden: true
# ---%%

const graphStaff = document.createElement("div")

var changePerDay = df.div(df.delta("staffPositive"),df.delta("date")).map((x)=>{return(x*60*24*60*1000)});
var changePerDayShifted = df.div(df.delta("staffPositive"),df.delta("date")).map((x)=>{return(x*60*24*60*1000)});
changePerDayShifted.unshift(1);
var percentChangePerDay = df.div(df.sub(changePerDayShifted,changePerDay),changePerDay).map((x)=>{return (x*100)});

var trace1 = {
  x: df.data.date,
  y: changePerDay,
  name: 'Change in Number of Staff Positive',
  type: 'scatter'
};


var trace2 = {
  x: df.data.date,
  y: percentChangePerDay,
  name: 'Percent Change in Number of Staff Positive',
  yaxis: 'y2',
  type: 'scatter'
};


var data = [trace1, trace2];

var layout = {
  autosize: true,
  width: graphWidth,
  height: graphHeight,
  title: 'ASU Staff Covid Change per Day',
  yaxis: {title: 'Change'},
  yaxis2: {
    title: 'Percent Change',
    titlefont: {color: 'rgb(148, 103, 189)'},
    tickfont: {color: 'rgb(148, 103, 189)'},
    overlaying: 'y',
    side: 'right'
  }
};

Plotly.newPlot(graphStaff, data, layout);

graphStaff
# %%--- [javascript]
# properties:
#   run_on_load: true
#   top_hidden: true
# ---%%

const graphStudent = document.createElement("div")

var trace1 = {
  x: df.data.date,
  y: df.data.studentPositive ,
  name: 'Number of Student Positive',
  type: 'scatter'
};


var trace2 = {
  x: df.data.date,
  y: df.data.studentPercent.map((x)=>x*100),
  name: 'Percentage of Student Positive',
  yaxis: 'y2',
  type: 'scatter'
};


var data = [trace1, trace2];

var layout = {
  autosize: true,
  width: graphWidth,
  height: graphHeight,
  title: 'ASU Student Covid',
  yaxis: {title: 'Number of Student Positive'},
  yaxis2: {
    title: 'Percentage of Student Positive',
    titlefont: {color: 'rgb(148, 103, 189)'},
    tickfont: {color: 'rgb(148, 103, 189)'},
    overlaying: 'y',
    side: 'right'
  }
};

Plotly.newPlot(graphStudent, data, layout);

graphStudent
# %%--- [javascript]
# properties:
#   run_on_load: true
#   top_hidden: true
# ---%%

const graphStaff = document.createElement("div")

var changePerDay = df.div(df.delta("studentPositive"),df.delta("date")).map((x)=>{return(x*60*24*60*1000)});
var changePerDayShifted = df.div(df.delta("studentPositive"),df.delta("date")).map((x)=>{return(x*60*24*60*1000)});
changePerDayShifted.unshift(1);
var percentChangePerDay = df.div(df.sub(changePerDayShifted,changePerDay),changePerDay).map((x)=>{return (x*100)});

console.log();
var trace1 = {
  x: df.data.date,
  y: changePerDay,
  name: 'Change in Number of Student Positive',
  type: 'scatter'
};


var trace2 = {
  x: df.data.date,
  y: percentChangePerDay,
  name: 'Percent Change in Number of Student Positive',
  yaxis: 'y2',
  type: 'scatter'
};


var data = [trace1, trace2];

var layout = {
  autosize: true,
  width: graphWidth,
  height: graphHeight,
  title: 'ASU Student Covid Change per Day',
  yaxis: {title: 'Change'},
  yaxis2: {
    title: 'Percentage Change',
    titlefont: {color: 'rgb(148, 103, 189)'},
    tickfont: {color: 'rgb(148, 103, 189)'},
    overlaying: 'y',
    side: 'right'
  }
};

Plotly.newPlot(graphStaff, data, layout);

graphStaff
# %% [markdown]
### COVID-19 Download Data 

Click the button to download 
# %%--- [javascript]
# properties:
#   run_on_load: true
# ---%%
function downloadCSV(){
	var hiddenElement = document.createElement('a');
    hiddenElement.href = 'data:text/csv;charset=utf-8,' + encodeURI(df.csv);
    hiddenElement.target = '_blank';
    
    //provide the name for the CSV file to be downloaded
    hiddenElement.download = 'ASU_Covid_Data.csv';
    hiddenElement.click();
  }

var downloadButton = document.createElement("button");
downloadButton.innerHTML = "Download CSV"
downloadButton.onclick = downloadCSV

downloadButton

{% endNotebook %}