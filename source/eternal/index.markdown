---
layout: page
title: "Sealed helper"
date: 2018-09-07
---

This page is intended to help you sort through an Eternal Card Game
sealed pool. It uses the TDC draft tier lists as a base value for cards
in a limited format like sealed.

1. Choose your tier threshold.
2. Paste your exported pool into the text box.
3. Click sort.

The cards in your pool with values exceeding your threshold will be
output at the bottom. If a card has a value above 4.0, it will be
<strong>bold</strong>. If a card has a value above 3.5, it will be
<em>italicized</em>. You should be able to import the output text
into Eternal to further work with your pool and determine which of
your factions is strongest.

NOTE: At the current time, influence strangers and banners are not
included in the card value pool. You'll need to include these manually.

<select id="threshold">
  <option value="4.5">4.5 - bomb, dominates game if unanswered</option>
  <option value="4.0">4.0 - high impact card generating value or tempo</option>
  <option value="3.5">3.5 - premium card, pulls you into a color</option>
  <option value="3.0" selected="true">3.0 - good playable, almost always makes cut</option>
  <option value="2.5">2.5 - solid playable, rarely cut</option>
  <option value="2.0">2.0 - good filler, sometimes gets cut</option>
</select>
<input type="button" value="Sort" onclick="sort()"></input>

<textarea cols="60" rows="20" id="pool"></textarea>

<h4>Sorted pool</h4>

<div id="result">
</div>

<script type="text/javascript">
var cardsAndValues = []
$.getJSON("/eternal/cards-and-values.json", function(data) {
    $.each(data, function(index, value) {
        cardsAndValues.push(value)
    })
})

function sort() {
    $("#result").empty()
    pool = []
    lines = $('#pool').val().trim().split("\n");
    $.each(lines, function() {
        values = this.split("(");
        numberOfAndName = values[0]
        numberOfCards = parseInt(numberOfAndName.charAt(0))
        cardName = numberOfAndName.substring(2).trim()
        setAndNumber = values[1]
        result = scanf(setAndNumber, "Set%d #%d)")
        card = {
            numberOfCards: numberOfCards,
            name: cardName,
            set: result[0],
            cardNumber: result[1]
        }
        for (i = 0; i < numberOfCards; i++) {
            pool.push(card)
        }
    })

    valuedPool = []
    $.each(pool, function(index, value) {
        card = findCard(value)
        if (!$.isEmptyObject(card)) {
            valuedPool.push(card)
        }
    })

    valuedPool.sort(SortByValue).reverse()

    threshold = $("#threshold").val()
    $.each(valuedPool, function(index, value) {
        if (value.LimitedValue >= threshold) {
            output = "1 " + value.Name + " (Set" + value.SetNumber + " #" + value.EternalID + ")"
            if (value.LimitedValue >= 4.0) {
                output = "<strong>" + output + "</strong>"
            } else if (value.LimitedValue >= 3.0) {
                output = "<em>" + output + "</em>"
            }
            $("#result").append(output + " <br/>")
        }
    })
}

function SortByValue(a, b) {
    return a.LimitedValue < b.LimitedValue ? -1 : a.LimitedValue > b.LimitedValue ? 1 : 0
}

function findCard(card) {
    result = {}
    $.each(cardsAndValues, function(index, value) {
        if (card.set == value.SetNumber &&
            card.cardNumber == value.EternalID) {
            result = value
        }
    })
    return result
}

function scanf(text,pattern){
    if (text == pattern) return true;
    var result = [];    // array for pattern result
    var i = 0;            // text index
    var j = 0;            // pattern index
    while (i < text.length && j < pattern.length){
        var p = substr(pattern,j,j+2); 
        var c = text[i];                
        var c2 = pattern[j];           
        if (p == "%d"){            
        // pattern says next is a number:
            var z = parseInt(substr(text,i,text.length));
            if (z == NaN) return false;
            result[result.length] = z;
            i += z.toString().length;
            j += 2;
        }
        else if (p == "%c"){    
        // pattern says next is a single character:
            result[result.length] = c;
            i++;
            j += 2;
        }
        else if (p == "%s"){    
        // pattern says next is a string:
            var end = "";
            if (j+2 < pattern.length) end = pattern[j+2];
            if (end.length == 0){
                result[result.length] = substr(text,i,text.length);
                i = text.length;
                j = pattern.length;
            }
            else if (end == '%'){    
            // This is an ERROR I need to fix!!!
                alert("[*] %s followed by pattern (eg. %d) causes an error!");
                return false;
            }
            else {
                var str = "";
                for (;i<text.length && text[i]!=end;i++){
                    str += text[i];
                }
                result[result.length] = str;
                j += 2;                            
            }
        }
        else if (c == c2){        
        // pattern says next char's should be equal:
            i++;
            j++;
        }
        else {                    
        // else the text doesn't fit to the pattern:
            return false;
        }
    }
    if (i == text.length && j == pattern.length){
        // if we scanned EVERYTHING:
        return result;            
    }
    else {
        // if not -> FALSE:
        return false;            
    }
}

function substr(str,i,j){
    var s = "";
    if (i < 0 || j < 0 || i > j) return false;
    for (var k=i;k<str.length && k<j;k++){
        s += str[k];
    }
    return s;
}
</script>
