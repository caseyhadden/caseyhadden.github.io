---
layout: page
title: "Sealed helper"
date: 2018-09-07
---

TL;DR: [Just take me to the tool](#tool)

This page is intended to help you sort through an Eternal Card Game
sealed pool. It uses the TDC draft tier lists as a base value for cards
in a limited format like sealed.

1. Choose your tier threshold.
2. Paste your exported pool into the text box.
3. Click sort.

The cards in your pool with values exceeding your threshold will be
output at the bottom. If a card has a value above 4.0, it will be
<strong>bold</strong>. If a card has a value above 3.0, it will be
<em>italicized</em>. You should be able to import the output text
into Eternal to further work with your pool and determine which of
your factions is strongest.

If you're seeing cards show up as <strike>struck out</strike> at the
bottom of the results, please let me know. I think a few are missing
from the value data. It could also be that the card was not in one
of the tier lists.

The core of how I use the tool is fairly simple:

1. Sort the pool
2. Try to play as many 4.0+ cards as I can
3. Balance #2 against trying to play as few <2.0 cards as I can
4. Try to ensure the influence requirements from [Flash2351's Draft 101 article](https://www.a-space-games.com/drafting-101-part-2-building-the-deck)
   are met.

If you want a longer form description, check out [this example](/eternal/sealed-example).
Feedback of all kinds is welcome. Let me know, if you think the tool
is great or terrible. Perhaps I'd be better off building decks through
a different process; let me know. Or anything else. We're all trying
to improve.

1. Email me@caseyhadden.com
2. Send me a DM on discord Dendroaspis#2087.

DISCLAIMERS:

1. Tier lists are a place to start, not an all encompassing set of rules.
2. Draft and sealed are different formats and card values will
sometimes be different between the two. I think the draft tier list
gets you 'in the ballpark' with human intelligence required from there.
3. Context matters. Cards can have a good rating, but be dependent on your
deck - renown units want pump spells and weapons, Slushdumper wants additional
yetis, etc. These criteria are often important to a card's overall value. In
draft, you can adjust your valuation criteria for these types of synergies.
However, in sealed you only have whatever is in your packs. Keep these type
of contextual situations in mind when building your deck.
4. Sealed adds an extra dimension with new packs each week. For the best
result, it is likely a good option to completely re-evaluate your pool
from scratch each week vs. assuming you should just build onto what you
had before.

<a id="tool"></a>Included Factions: <input type="checkbox" id="fire" value="F" checked><label for="fire">Fire</label>
<input type="checkbox" id="time" value="T" checked><label for="time">Time</label>
<input type="checkbox" id="justice" value="J" checked><label for="justice">Justice</label>
<input type="checkbox" id="primal" value="P" checked><label for="primal">Primal</label>
<input type="checkbox" id="shadow" value="S" checked><label for="shadow">Shadow</label>
<br/>Threshold:
<select id="threshold">
  <option value="4.5">&gt;= 4.5 - bomb, dominates game if unanswered</option>
  <option value="4.0">&gt;= 4.0 - high impact card generating value or tempo</option>
  <option value="3.5">&gt;= 3.5 - premium card, pulls you into a color</option>
  <option value="3.0" selected="true">&gt;= 3.0 - good playable, almost always makes cut</option>
  <option value="2.5">&gt;= 2.5 - solid playable, rarely cut</option>
  <option value="2.0">&gt;= 2.0 - good filler, sometimes gets cut</option>
  <option value="1.5">&gt;= 1.5 - filler, gets cut half the time</option>
  <option value="1.0">&gt;= 1.0 - bad filler, gets cut most of the time</option>
  <option value="0.5">&gt;= 0.5 - very low-end playables, sideboard material</option>
  <option value="0.0">&gt;= 0.0 - unplayable</option>
</select>
<input type="button" value="Sort" onclick="sort()"></input>


<strong>16Jun2019 - This tool now uses the latest TDC tier list for Dark Frontier.</strong>

<textarea cols="60" rows="20" id="pool"></textarea>

<h4>Sorted pool <input type="button" value="Toggle details" onclick="toggleExtra()"></input></h4>

<div id="result">
</div>

<br/>
<strong>The charts below are heavily inspired by the
<a href="https://eternalsealed.surge.sh/">Eternal Sealed Analyzer</a>.</strong>

<canvas id="monoChart">
</canvas>

<br/>
<canvas id="dualChart">
</canvas>

<script type="text/javascript">
var cardsAndValues = []
$.getJSON("/eternal/cards-and-values.json", function(data) {
    $.each(data, function(index, value) {
        cardsAndValues.push(value)
    })
})

var monoChart, dualChart

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
    notFoundPool = []
    $.each(pool, function(index, value) {
        card = findCard(value)
        if (!$.isEmptyObject(card)) {
            valuedPool.push(card)
        } else {
          notFoundPool.push(value)
        }
    })

    valuedPool.sort(SortByValue).reverse()

    requestedInfluence = []
    $.each($("input:checked"), function(index, value) {
      requestedInfluence.push(value.value)
    })

    let fire = {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Fire",
    }, time =  {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Time",
    }, justice =  {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Justice",
    }, primal = {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Primal",
    }, shadow = {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Shadow",
    }, rakano = {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Rakano",
    }, argenport = {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Argenport",
    }, hooru = {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Hooru",
    }, combrei = {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Combrei",
    }, stonescar = {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Stonescar",
    }, skycrag = {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Skycrag",
    }, praxis = {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Praxis",
    }, feln = {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Feln",
    }, xenan = {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Xenan",
    }, elysian = {
      unit: 0,
      spell: 0,
      attachment: 0,
      name: "Elysian",
    };

    threshold = $("#threshold").val()
    tableContent = "<table id='sort_results'>"
    $.each(valuedPool, function(index, value) {
        cardInfluence = makeUnique(value.Influence)
        influenceDiff = cardInfluence.filter(x => !requestedInfluence.includes(x))
        influenceString = cardInfluence.join("")
        if (value.LimitedValue >= threshold && influenceDiff.length == 0) {
            output = "1 " + value.Name + " (Set" + value.SetNumber + " #" + value.EternalID + ")"
            if (value.LimitedValue >= 4.0) {
                output = "<strong>" + output + "</strong>"
            } else if (value.LimitedValue >= 3.0) {
                output = "<em>" + output + "</em>"
            }
            tableContent += "<tr>"
            tableContent += "<td class='card_data'>" + output + "</td>"
            tableContent += "<td style='padding-left: 10px' class='influence_data'>" + value.Influence + "</td>"
            tableContent += "<td style='padding-left: 10px' class='value_data'>" + value.LimitedValue + "</td>"
            tableContent += "</tr>"
            // update our counts for charts
            switch (influenceString) {
              case "F":
                updateCounts(fire, value)
                break;
              case "T":
                updateCounts(time, value)
                break;
              case "J":
                updateCounts(justice, value)
                break;
              case "P":
                updateCounts(primal, value)
                break;
              case "S":
                updateCounts(shadow, value)
                break;
              case "FJ":
                updateCounts(rakano, value)
                break;
              case "JS":
                updateCounts(argenport, value)
                break;
              case "JP":
                updateCounts(hooru, value)
                break;
              case "TJ":
                updateCounts(combrei, value)
                break;
              case "FS":
                updateCounts(stonescar, value)
                break;
              case "FP":
                updateCounts(skycrag, value)
                break;
              case "FT":
                updateCounts(praxis, value)
                break;
              case "PS":
                updateCounts(feln, value)
                break;
              case "TS":
                updateCounts(xenan, value)
                break;
              case "TP":
                updateCounts(elysian, value)
                break;
            }
        }
    })
    tableContent += "</table>"
    $("#result").append(tableContent)
    toggleExtra() // hide to start

    if (monoChart) {
      monoChart.destroy()
    }
    monoChart = new Chart($("#monoChart"), {
        type: "bar",
        data: {
            labels: ["Fire", "Time", "Justice", "Primal", "Shadow"],
            datasets: [
                {
                    label: "Unit",
                    data: [
                        fire.unit,
                        time.unit,
                        justice.unit,
                        primal.unit,
                        shadow.unit,
                    ],
                    backgroundColor: [
                        "rgba(255,99,132,0.6)",
                        "rgba(255, 206, 86, 0.6)",
                        "rgba(75, 192, 192, 0.6)",
                        "rgba(54, 162, 235, 0.6)",
                        "rgba(153, 102, 255, 0.6)",
                    ],
                    borderColor: [
                        "rgba(255,99,132,1)",
                        "rgba(255, 206, 86, 1)",
                        "rgba(75, 192, 192, 1)",
                        "rgba(54, 162, 235, 1)",
                        "rgba(153, 102, 255, 1)",
                    ],
                    borderWidth: 1,
                },
                {
                    label: "Spell",
                    data: [
                        fire.spell,
                        time.spell,
                        justice.spell,
                        primal.spell,
                        shadow.spell,
                    ],
                    backgroundColor: [
                        "rgba(255,99,132,0.4)" /*R*/,
                        "rgba(255, 206, 86, 0.4)" /*W*/,
                        "rgba(75, 192, 192, 0.4)" /*G*/,
                        "rgba(54, 162, 235, 0.4)" /*U*/,
                        "rgba(153, 102, 255, 0.4)" /*B*/,
                    ],
                    borderColor: [
                        "rgba(255,99,132,1)",
                        "rgba(255, 206, 86, 1)",
                        "rgba(75, 192, 192, 1)",
                        "rgba(54, 162, 235, 1)",
                        "rgba(153, 102, 255, 1)",
                    ],
                    borderWidth: 1,
                },
                {
                    label: "Attachment",
                    data: [
                        fire.attachment,
                        time.attachment,
                        justice.attachment,
                        primal.attachment,
                        shadow.attachment,
                    ],
                    backgroundColor: [
                        "rgba(255,99,132,0.2)" /*R*/,
                        "rgba(255, 206, 86, 0.2)" /*W*/,
                        "rgba(75, 192, 192, 0.2)" /*G*/,
                        "rgba(54, 162, 235, 0.2)" /*U*/,
                        "rgba(153, 102, 255, 0.2)" /*B*/,
                    ],
                    borderColor: [
                        "rgba(255,99,132,1)",
                        "rgba(255, 206, 86, 1)",
                        "rgba(75, 192, 192, 1)",
                        "rgba(54, 162, 235, 1)",
                        "rgba(153, 102, 255, 1)",
                    ],
                    borderWidth: 1,
                },
            ],
        },
        options: {
            legend: {
                display: false,
            },
            scales: {
                xAxes: [
                    {
                        stacked: true,
                        ticks: {
                            autoSkip: false,
                        },
                    },
                ],
                yAxes: [
                    {
                        stacked: true,
                        ticks: {
                            beginAtZero: true,
                        },
                    },
                ],
            },
        },
    });

    if (dualChart) {
      dualChart.destroy()
    }
    dualChart = new Chart($("#dualChart"), {
        type: "bar",
        data: {
            labels: [
                "Rakano",
                "Argenport",
                "Hooru",
                "Combrei",
                "Stonescar",
                "Skycrag",
                "Praxis",
                "Feln",
                "Xenan",
                "Elysian",
            ],
            datasets: [
                {
                    label: "Units - Mono 1",
                    data: [
                        fire.unit,
                        justice.unit,
                        justice.unit,
                        justice.unit,
                        fire.unit,
                        fire.unit,
                        fire.unit,
                        primal.unit,
                        shadow.unit,
                        primal.unit,
                    ],
                    backgroundColor: [
                        "rgba(255,99,132,0.6)", //Fire
                        "rgba(75, 192, 192, 0.6)", //Justice
                        "rgba(75, 192, 192, 0.6)",
                        "rgba(75, 192, 192, 0.6)",
                        "rgba(255,99,132,0.6)",
                        "rgba(255,99,132,0.6)",
                        "rgba(255,99,132,0.6)",
                        "rgba(54, 162, 235, 0.6)", //Primal
                        "rgba(153, 102, 255, 0.6)", //Shadow
                        "rgba(54, 162, 235, 0.6)",
                    ],
                    borderColor: [
                        "rgba(255,99,132,1)",
                        "rgba(75, 192, 192, 1)",
                        "rgba(75, 192, 192, 1)",
                        "rgba(75, 192, 192, 1)",
                        "rgba(255,99,132,1)",
                        "rgba(255,99,132,1)",
                        "rgba(255,99,132,1)",
                        "rgba(54, 162, 235,14)",
                        "rgba(153, 102, 255, 1)",
                        "rgba(54, 162, 235,14)",
                    ],
                    borderWidth: 1,
                },
                {
                    label: "Units - Mono 2",
                    data: [
                        justice.unit,
                        shadow.unit,
                        primal.unit,
                        time.unit,
                        shadow.unit,
                        primal.unit,
                        time.unit,
                        shadow.unit,
                        time.unit,
                        time.unit,
                    ],
                    backgroundColor: [
                        "rgba(75, 192, 192, 0.6)" /*G*/,
                        "rgba(153, 102, 255,0.6)" /*B*/,
                        "rgba(54, 162, 235, 0.6)" /*U*/,
                        "rgba(255, 206, 86, 0.6)" /*W*/,
                        "rgba(153, 102, 255,0.6)" /*B*/,
                        "rgba(54, 162, 235, 0.6)" /*U*/,
                        "rgba(255, 206, 86, 0.6)" /*W*/,
                        "rgba(153, 102, 255,0.6)" /*B*/,
                        "rgba(255, 206, 86, 0.6)" /*W*/,
                        "rgba(255, 206, 86, 0.6)" /*W*/,
                    ],
                    borderColor: [
                        "rgba(75, 192, 192, 1)" /*G*/,
                        "rgba(153, 102, 255, 1)" /*B*/,
                        "rgba(54, 162, 235, 1)" /*U*/,
                        "rgba(255, 206, 86, 1)" /*W*/,
                        "rgba(153, 102, 255, 1)" /*B*/,
                        "rgba(54, 162, 235, 1)" /*U*/,
                        "rgba(255, 206, 86, 1)" /*W*/,
                        "rgba(153, 102, 255, 1)" /*B*/,
                        "rgba(255, 206, 86, 1)" /*W*/,
                        "rgba(255, 206, 86,12)" /*W*/,
                    ],
                    borderWidth: 1,
                },
                {
                    label: "Spells - Mono 1",
                    data: [
                        fire.spell,
                        justice.spell,
                        justice.spell,
                        justice.spell,
                        fire.spell,
                        fire.spell,
                        fire.spell,
                        primal.spell,
                        shadow.spell,
                        primal.spell,
                    ],
                    backgroundColor: [
                        "rgba(255,99,132,0.4)", //Fire
                        "rgba(75, 192, 192, 0.4)", //Justice
                        "rgba(75, 192, 192, 0.4)",
                        "rgba(75, 192, 192, 0.4)",
                        "rgba(255,99,132,0.4)",
                        "rgba(255,99,132,0.4)",
                        "rgba(255,99,132,0.4)",
                        "rgba(54, 162, 235, 0.4)", //Primal
                        "rgba(153, 102, 255, 0.4)", //Shadow
                        "rgba(54, 162, 235, 0.4)",
                    ],
                    borderColor: [
                        "rgba(255,99,132,1)",
                        "rgba(75, 192, 192, 1)",
                        "rgba(75, 192, 192, 1)",
                        "rgba(75, 192, 192, 1)",
                        "rgba(255,99,132,1)",
                        "rgba(255,99,132,1)",
                        "rgba(255,99,132,1)",
                        "rgba(54, 162, 235,14)",
                        "rgba(153, 102, 255, 1)",
                        "rgba(54, 162, 235,14)",
                    ],
                    borderWidth: 1,
                },
                {
                    label: "Spells - Mono 2",
                    data: [
                        justice.spell,
                        shadow.spell,
                        primal.spell,
                        time.spell,
                        shadow.spell,
                        primal.spell,
                        time.spell,
                        shadow.spell,
                        time.spell,
                        time.spell,
                    ],
                    backgroundColor: [
                        "rgba(75, 192, 192, 0.4)" /*G*/,
                        "rgba(153, 102, 255,0.4)" /*B*/,
                        "rgba(54, 162, 235, 0.4)" /*U*/,
                        "rgba(255, 206, 86, 0.4)" /*W*/,
                        "rgba(153, 102, 255,0.4)" /*B*/,
                        "rgba(54, 162, 235, 0.4)" /*U*/,
                        "rgba(255, 206, 86, 0.4)" /*W*/,
                        "rgba(153, 102, 255,0.4)" /*B*/,
                        "rgba(255, 206, 86, 0.4)" /*W*/,
                        "rgba(255, 206, 86, 0.4)" /*W*/,
                    ],
                    borderColor: [
                        "rgba(75, 192, 192, 1)" /*G*/,
                        "rgba(153, 102, 255, 1)" /*B*/,
                        "rgba(54, 162, 235, 1)" /*U*/,
                        "rgba(255, 206, 86, 1)" /*W*/,
                        "rgba(153, 102, 255, 1)" /*B*/,
                        "rgba(54, 162, 235, 1)" /*U*/,
                        "rgba(255, 206, 86, 1)" /*W*/,
                        "rgba(153, 102, 255, 1)" /*B*/,
                        "rgba(255, 206, 86, 1)" /*W*/,
                        "rgba(255, 206, 86,12)" /*W*/,
                    ],
                    borderWidth: 1,
                },
                {
                    label: "Attachments - Mono 1",
                    data: [
                        fire.attachment,
                        justice.attachment,
                        justice.attachment,
                        justice.attachment,
                        fire.attachment,
                        fire.attachment,
                        fire.attachment,
                        primal.attachment,
                        shadow.attachment,
                        primal.attachment,
                    ],
                    backgroundColor: [
                        "rgba(255,99,132,0.2)", //Fire
                        "rgba(75, 192, 192, 0.2)", //Justice
                        "rgba(75, 192, 192, 0.2)",
                        "rgba(75, 192, 192, 0.2)",
                        "rgba(255,99,132,0.2)",
                        "rgba(255,99,132,0.2)",
                        "rgba(255,99,132,0.2)",
                        "rgba(54, 162, 235, 0.2)", //Primal
                        "rgba(153, 102, 255, 0.2)", //Shadow
                        "rgba(54, 162, 235, 0.2)",
                    ],
                    borderColor: [
                        "rgba(255,99,132,1)",
                        "rgba(75, 192, 192, 1)",
                        "rgba(75, 192, 192, 1)",
                        "rgba(75, 192, 192, 1)",
                        "rgba(255,99,132,1)",
                        "rgba(255,99,132,1)",
                        "rgba(255,99,132,1)",
                        "rgba(54, 162, 235,14)",
                        "rgba(153, 102, 255, 1)",
                        "rgba(54, 162, 235,14)",
                    ],
                    borderWidth: 1,
                },
                {
                    label: "Attachments - Mono 2",
                    data: [
                        justice.attachment,
                        shadow.attachment,
                        primal.attachment,
                        time.attachment,
                        shadow.attachment,
                        primal.attachment,
                        time.attachment,
                        shadow.attachment,
                        time.attachment,
                        time.attachment,
                    ],
                    backgroundColor: [
                        "rgba(75, 192, 192, 0.2)" /*G*/,
                        "rgba(153, 102, 255,0.2)" /*B*/,
                        "rgba(54, 162, 235, 0.2)" /*U*/,
                        "rgba(255, 206, 86, 0.2)" /*W*/,
                        "rgba(153, 102, 255,0.2)" /*B*/,
                        "rgba(54, 162, 235, 0.2)" /*U*/,
                        "rgba(255, 206, 86, 0.2)" /*W*/,
                        "rgba(153, 102, 255,0.2)" /*B*/,
                        "rgba(255, 206, 86, 0.2)" /*W*/,
                        "rgba(255, 206, 86, 0.2)" /*W*/,
                    ],
                    borderColor: [
                        "rgba(75, 192, 192, 1)" /*G*/,
                        "rgba(153, 102, 255, 1)" /*B*/,
                        "rgba(54, 162, 235, 1)" /*U*/,
                        "rgba(255, 206, 86, 1)" /*W*/,
                        "rgba(153, 102, 255, 1)" /*B*/,
                        "rgba(54, 162, 235, 1)" /*U*/,
                        "rgba(255, 206, 86, 1)" /*W*/,
                        "rgba(153, 102, 255, 1)" /*B*/,
                        "rgba(255, 206, 86, 1)" /*W*/,
                        "rgba(255, 206, 86,12)" /*W*/,
                    ],
                    borderWidth: 1,
                },
                {
                    label: "Units - Dual",
                    data: [
                        rakano.unit,
                        argenport.unit,
                        hooru.unit,
                        combrei.unit,
                        stonescar.unit,
                        skycrag.unit,
                        praxis.unit,
                        feln.unit,
                        xenan.unit,
                        elysian.unit,
                    ],
                    backgroundColor: [
                        "rgba(75, 75, 75, 0.6)" /*Gray*/,
                        "rgba(75, 75, 75, 0.6)" /*Gray*/,
                        "rgba(75, 75, 75, 0.6)" /*Gray*/,
                        "rgba(75, 75, 75, 0.6)" /*Gray*/,
                        "rgba(75, 75, 75, 0.6)" /*Gray*/,
                        "rgba(75, 75, 75, 0.6)" /*Gray*/,
                        "rgba(75, 75, 75, 0.6)" /*Gray*/,
                        "rgba(75, 75, 75, 0.6)" /*Gray*/,
                        "rgba(75, 75, 75, 0.6)" /*Gray*/,
                        "rgba(75, 75, 75, 0.6)" /*Gray*/,
                    ],
                    borderColor: [
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                    ],
                    borderWidth: 1,
                },
                {
                    label: "Spells - Dual",
                    data: [
                        rakano.spell,
                        argenport.spell,
                        hooru.spell,
                        combrei.spell,
                        stonescar.spell,
                        skycrag.spell,
                        praxis.spell,
                        feln.spell,
                        xenan.spell,
                        elysian.spell,
                    ],
                    backgroundColor: [
                        "rgba(75, 75, 75, 0.4)" /*Gray*/,
                        "rgba(75, 75, 75, 0.4)" /*Gray*/,
                        "rgba(75, 75, 75, 0.4)" /*Gray*/,
                        "rgba(75, 75, 75, 0.4)" /*Gray*/,
                        "rgba(75, 75, 75, 0.4)" /*Gray*/,
                        "rgba(75, 75, 75, 0.4)" /*Gray*/,
                        "rgba(75, 75, 75, 0.4)" /*Gray*/,
                        "rgba(75, 75, 75, 0.4)" /*Gray*/,
                        "rgba(75, 75, 75, 0.4)" /*Gray*/,
                        "rgba(75, 75, 75, 0.4)" /*Gray*/,
                    ],
                    borderColor: [
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                    ],
                    borderWidth: 1,
                },
                {
                    label: "Attachments - Dual",
                    data: [
                        rakano.attachment,
                        argenport.attachment,
                        hooru.attachment,
                        combrei.attachment,
                        stonescar.attachment,
                        skycrag.attachment,
                        praxis.attachment,
                        feln.attachment,
                        xenan.attachment,
                        elysian.attachment,
                    ],
                    backgroundColor: [
                        "rgba(75, 75, 75, 0.2)" /*Gray*/,
                        "rgba(75, 75, 75, 0.2)" /*Gray*/,
                        "rgba(75, 75, 75, 0.2)" /*Gray*/,
                        "rgba(75, 75, 75, 0.2)" /*Gray*/,
                        "rgba(75, 75, 75, 0.2)" /*Gray*/,
                        "rgba(75, 75, 75, 0.2)" /*Gray*/,
                        "rgba(75, 75, 75, 0.2)" /*Gray*/,
                        "rgba(75, 75, 75, 0.2)" /*Gray*/,
                        "rgba(75, 75, 75, 0.2)" /*Gray*/,
                        "rgba(75, 75, 75, 0.2)" /*Gray*/,
                    ],
                    borderColor: [
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                        "rgba(75, 75, 75, 1)" /*Gray*/,
                    ],
                    borderWidth: 1,
                },
            ],
        },
        options: {
            legend: {
                display: false,
            },
            scales: {
                xAxes: [
                    {
                        stacked: true,
                        ticks: {
                            autoSkip: false,
                        },
                    },
                ],
                yAxes: [
                    {
                        stacked: true,
                        ticks: {
                            beginAtZero: true,
                        },
                    },
                ],
            },
        },
    });

    $.each(notFoundPool, function(index, value) {
      output = "1 " + value.name + " (Set" + value.set + " #" + value.cardNumber + ")"
      $("#result").append("<strike>" + output + "</strike><br/>")
    })
}

function updateCounts(item, value) {
    switch (value.Type) {
      case "Unit":
        item.unit++
        break
      case "Spell":
      case "Fast Spell":
        item.spell++
        break
      case "Curse":
      case "Relic":
      case "Cursed Relic":
      case "Relic Weapon":
      case "Weapon":
        item.attachment++
        break
      default:
        console.log(value.Type)
    }
}

function toggleExtra() {
    tbl = $("#sort_results")
    console.log(tbl)
    influenceColumn = tbl.find(".influence_data")
    console.log(influenceColumn)
    influenceColumn.toggle()
    tbl.find(".value_data").toggle()
}

function makeUnique(str) {
  uniq = String.prototype.concat(...new Set(str))
  return uniq.split("")
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

