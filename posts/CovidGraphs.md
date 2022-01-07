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
var proxyUrl = "https://api.allorigins.win/get?url=";
var graphWidth = 900;
var graphHeight = 500;


var data = await fetch(proxyUrl+asuCovidUrl);
var htmlData = await data.json();
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
      	.match(/\d+/g)
      	.map((x)=>parseInt(x))
  
      var intArrayStudent = currentPositive.body.getElementsByTagName("li")[1].innerHTML
      	.replace(/&nbsp;/g, ' ')
      	.replace(",","")
      	.replace("*","")
      	.match(/\d+/g)
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
      this.date = [];
      this.staffPositive = [];
      this.staffTotal = [];
      this.staffPercent = [];
      this.studentPositive = [];
      this.studentTotal = [];
      this.studentPercent = [];
	  this.csv = "data,staffPositive,staffTotal,staffPercent,studentPositive,studentTotal,studentPercent \\n";
      
      for(var i = 0; i < entryArray.length; i++){
        if(entryArray[i].currentPositive!=null){
          this.date.push(entryArray[i].date);
          this.staffPositive.push(entryArray[i].currentPositive.staffPositive);
          this.staffTotal.push(entryArray[i].currentPositive.staffTotal);
          this.staffPercent.push(entryArray[i].currentPositive.staffPercent);
          this.studentPositive.push(entryArray[i].currentPositive.studentPositive);
          this.studentTotal.push(entryArray[i].currentPositive.studentTotal);
          this.studentPercent.push(entryArray[i].currentPositive.studentPercent);
          this.csv = this.csv + entryArray[i].date.toString()+","+entryArray[i].currentPositive.staffPositive.toString()+"," + entryArray[i].currentPositive.staffTotal.toString()+","+entryArray[i].currentPositive.staffPercent.toString()+","+entryArray[i].currentPositive.studentPositive.toString()+","+entryArray[i].currentPositive.studentTotal.toString()+","+entryArray[i].currentPositive.studentPercent.toString()+"\\n"
          }
      }
    }
	dataGen(set1,set2){
      data = [];
      for(var i = 0; i < set1.length; i ++){
        data.push({x:set1[i],y:set2[i]})
      }
      return data; 
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
var doc = parser.parseFromString(htmlData.contents, "text/html");

//split data by pannels 
var docPannels = doc.getElementsByClassName("panel"); 

//iterate over 
var docPannelsArray = Object.values(docPannels);

var dataArray = []
for(var i = 0; i<docPannelsArray.length; i++){
	//extract date 
	dataArray.push(new DataEntry(docPannelsArray[i]));
}

var df = new DataFrame(dataArray)

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
  x: df.date,
  y: df.staffPositive ,
  name: 'Number of Staff Positive',
  type: 'scatter'
};


var trace2 = {
  x: df.date,
  y: df.staffPercent.map((x)=>x*100),
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

const graphStudent = document.createElement("div")

var trace1 = {
  x: df.date,
  y: df.studentPositive ,
  name: 'Number of Student Positive',
  type: 'scatter'
};


var trace2 = {
  x: df.date,
  y: df.studentPercent.map((x)=>x*100),
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
# %% [markdown]
### COVID-19 Download Data 

Click the button to download 
# %% [javascript]
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