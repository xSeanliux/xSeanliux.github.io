---
title: Gulsmi - Hittite Typesetting!

draft: false
---

<small> The sign images are from Prof. Shosted at UIUC, who extracted the images from the *Hethitische Zeichenlexikon*. 

:camera: Yazılıkaya, source: [World Pilgrimage Guide](https://sacredsites.com/middle_east/turkey/yazilikaya.html)
</small> 

{{< lead >}} *Gulsmi* means "I write" in Hittite. This is a small webapp that aims to let you do the same from the comfort of your home without the hassle of having to get reeds (or chopsticks) and clay! {{</lead >}}


<div id="mainframe">
    <div id="displayFrame">
        <span class="signWrapper"> </span>
    </div>
    <div id="inputForm">
        <div id="inputSpan">
            <input type="text" placeholder = "Enter transliteration here!" id="inputBox"/>
        </div>
        <div id="selectionHelp">
            <div id = "selectionContent"></div>
        </div>
    </div>
</div>

**Instructions**: Type your Hittite transliteration below, separated by dashes and without accents. Spaces aren't currently supported, so this is more just for singular words. Disambiguation for different renderings of the same phonetic input is rudimentarily supported by having you type an index after the symbol (e.g. *ze1*, *ze2*; *ek*, *ek2*). For example, you can copy-paste the below into the box below: 

* *DINGIR-ISTAR* the goddess Ishtar
* *ag-ga-a-at-ta-ar* death, ruin
* *Il-li-nu-wa* the spelling for [Illinois](http://faculty.las.illinois.edu/rshosted/ne%C5%A1ili.html) in Hittite

**Known Issues**
* There are quite a few missing signs, especially some Sumerograms/Akkadograms such as EZEN (without which we couldn't write about the festival of [AN.TAH.ŠUM](http://faculty.las.illinois.edu/rshosted/docs/Festival%20of%20AN-TAH-SUM.pdf)!)
* A lack of a "space bar" which isn't so conductive to sentences (*nu-ta*)
* Since signs have all been resized to the same height (72px), some flatter signs are out of proportion, including *be*, *hal*, *man*, and their variants. The glyph *as* has been patched, but the others are WIP.
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.6.4/jquery.min.js"></script>
<!-- <script src="/js/hittite_typesetting.js"> -->

<style>
#displayFrame {
overflow: auto;
  background-color: #fafafa;
  padding: 20px;
  /* margin-top: 5px;
  margin-bottom: 5px; */
  margin: auto;
  min-height: 110px;
  width: 100%;
  /* display: inline-grid; */
  /* grid-template-columns: repeat(5, 1fr); */
  /* grid-column-gap: 5px; */
  /* grid-row-gap:   10px; */
}

#inputForm {
    margin-top: 10px;
    color: #3c424d;
    display: flex;
    align-items: stretch;
    width: 100%;
}

#inputSpan {
    overflow: auto;
    
    flex-grow: 4;
}

input {
    width: 100%;
}

#selectionHelp {
    background-color: #e0e0e0;
    display: flex;
    flex: 1 1 auto;
    align-items:center;
    justify-content:center;
}

#selectionContent {
    margin-left: 10px;
}
.signWrapper {
    /* max-width: 100%; */
    float: left;
}

</style>

<script>

function findSignName(str) {
    //format of str: <glyph reference name>[number = idx]
    const re = /^([a-z]+)([0-9]*)$/;
    console.log(re.exec(str));
    matches = []

    let matchRes = re.exec(str);
    if(matchRes == null) return null;
    let symbol = matchRes[1];
    let index  = parseInt(matchRes[2]); //one-based
    if(!index) index = 1;
    
    if(index <= 0) return null;

    // console.log("symbol = <" + symbol + ">");
    for(let x in signList) {
        pair = signList[x];
        if(pair[0] == symbol) {
            matches.push(pair[1]);
        }
    }
    // console.log("index = ", index, "match count = ", matches.length);
    if(matches.length < index) return null;
    
    return { symbol, matches, index };
}

function parse(str) {
    let splits = str.split('-');
    let result = [];
    console.log(splits.length, splits);
    for(var i = 0; i < splits.length; i++) {
        let findRes = findSignName(splits[i].toLowerCase());
        if(findRes === null) {
            // console.log("Not found ", splits[i].toLowerCase());
            return null;
        }
        let { symbol, matches, index } = findRes;
        $("#selectionContent").html(`${symbol}: ${matches.length} sign${matches.length > 1 ? 's' : ''} found`);
        result.push(matches[index - 1]);
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
        // console.log("not fnull!,", parseResult);
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