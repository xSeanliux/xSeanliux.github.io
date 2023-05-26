---
title: "*Gulsmi* - Hittite Typesetting! "

draft: false
---

The images were made by Prof. Shosted at UIUC - full credit goes to him. *Gulsmi* means "I write."

Instructions: type your Hittite transliteration below, separated by dashes and without accents. Spaces aren't currently supported, so this is more just for singular words. In addition, disambiguation for different renderings of the same phonetic input haven't been implemented yet (e.g. *ak* vs *ek* or *u* and *ú*). For example, you can copy-paste the below into the box below: 

* *dingir-istar* the goddess Ishtar
* *ag-ga-a-at-ta-ar* death, ruin
* *il-li-nu-wa* the spelling for [Illinois in Hittite](http://faculty.las.illinois.edu/rshosted/ne%C5%A1ili.html)
<div id="mainframe">
    <div id="displayFrame">
        <span class="signWrapper"> </span>
    </div>
    <div id="inputForm">
        <span id="inputSpan">
            <input type="text" placeholder = "Enter transliteration here!" id="inputBox"></input>
        </span>
    </div>
</div>

<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.6.4/jquery.min.js"></script>
<!-- <script src="/js/hittite_typesetting.js"> -->

<style>
#displayFrame {
    overflow: auto;
  background-color: white;
  padding: 20px;
  margin-top: 5px;
  margin-bottom: 5px;
  min-height: 110px;
  width: 100%;
  /* display: inline-grid; */
  /* grid-template-columns: repeat(5, 1fr); */
  /* grid-column-gap: 5px; */
  /* grid-row-gap:   10px; */
}

#inputScan {
    display: block;
}

input {
    width: 100%;
}

.signWrapper {
    /* max-width: 100%; */
    float: left;
}
</style>

<script>

function findSignName(str) {
    for(let x in signList) {
        pair = signList[x];
        if(pair[0] == str) return pair[1];
    }
    return null;
}

function parse(str) {
    let splits = str.split('-');
    let result = [];
    console.log(splits.length, splits);
    for(var i = 0; i < splits.length; i++) {
        fileName = findSignName(splits[i].toLowerCase());
        if(fileName === null) {
            // console.log("Not found ", splits[i].toLowerCase());
            return null;
        }
        result.push(fileName);
    }
    return result;
}
var signList;
$("#mainframe").ready(function() {
    $.get("/cuneiform_names.txt", function(data) {
        // console.log(data);
        signList = data.split('\n');
        for(var i = 0; i < signList.length; i++) signList[i] = signList[i].split('\t');
    }, "text");
});

$("#inputBox").on("input", function() {
    let inputStr = $(this).val();
    parseResult = parse(inputStr);
    if(parseResult) {
        console.log("not fnull!,", parseResult);
        $("#displayFrame").html("");
        for(let i in parseResult) {
            $("#displayFrame").append(`
<span class="signWrapper">
    <img src="/cuneiform_images/${parseResult[i]}" class="sign">
</span>
            `);
        }
    }
});



</script>