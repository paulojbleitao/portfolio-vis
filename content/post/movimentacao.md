---
title: "Movimentação no açude velho"
date: 2017-12-14T19:41:43-03:00
draft: false
---

<meta charset="utf-8">
<script src="https://d3js.org/d3.v4.min.js"></script>
O açude velho é um ponto importante em Campina Grande devido à sua localização; por se encontrar no centro da cidade, o fluxo de locomoção em seu entorno é consideravelmente formidável. Por conta disso, muita gente transita por ele todos os dias. Com isso, podemos compilar informações das pessoas que por lá transitam em determinados períodos do dia e chegar a conclusões interessantes.

<div class="container" id="container1">
    <svg class="myvis1" id="vis1" width="800" height="500"></svg>
</div>
<div class="container">
    <div id="dropdown1">Horário inicial: </div>
    <div id="dropdown2">Horário final: </div>
    <button class="btn-default" id="botao">
        Atualizar gráfico
    </button>
</div><br/><br/>


O gráfico acima mostra a quantidade de pessoas que passam em cada ponto observado ao longo do dia. Podemos verificar que, principalmente próximo às 8:00 e 14:00, o monumento dos burrinhos recebe mais visitantes que os outros pontos. Ademais, o Bob's e a estátua de Jackson do Pandeiro apresentam, aproximadamente, a mesma quantidade de transeuntes no decorrer do dia.

Podemos também observar outras características das pessoas, como o meio de transporte: motorizado (carros, ônibus etc.) e não motorizado (ciclistas e pedestres), tal como no gráfico a seguir:

<div class="container">
    <svg class="myvis2" id="vis2" width="800" height="500"></svg>
</div>

Neste gráfico, a linha representa a quantidade de pessoas motorizadas que passam pelo açude em determinado período do dia, e a área é a diferença entre locomoção motorizada e não motorizada. Quando a área é azul, há mais pessoas não motorizadas do que motorizadas e quando é verde, ocorre o contrário. Podemos perceber que no início da manhã há poucas pessoas motorizadas circulando, mas ao longo do dia, não surpreendentemente, há muitas pessoas motorizadas e a diferença é bem expressiva. Além disso, também podemos observar que no fim do dia, a discrepância se torna bem menor.

Somos também capazes de diferenciar as pessoas não motorizadas entre homem e mulher e comparar as categorias:

<div class="container">
    <svg class="myvis3" id="vis3" width="800" height="500"></svg>
</div>

É perceptível que, geralmente, o número de homens que passa pelo açude predomina sobre o de mulheres. Ademais, o volume de ciclistas e pedestres diminui bastante no meio do dia.

Com tudo isso, podemos ter uma melhor ideia das características das pessoas que frequentam o açude velho. 

<!-- vis 1 -->
<script>
"use strict"

function desenhaEixo(data, scale) {
    var svg = d3.select("#vis1"),
        margin = {top: 20, right: 20, bottom: 30, left: 50},
        width = +svg.attr("width") - margin.left - margin.right,
        height = +svg.attr("height") - margin.top - margin.bottom,
        g = svg.append("g").attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    var parseTime = d3.timeParse("%H:%M");

    var x = d3.scaleTime()
        .rangeRound([0, width])
        .domain(d3.extent(data, function(d) { return parseTime(d.key); }));

    var y = d3.scaleLinear()
        .rangeRound([height, 0])
        .domain(d3.extent(scale, function(d) { return d; }));

    var keys = ["Bob's", "Estátua de Jackson", "Monumento dos Burrinhos"];
    
    var z = d3.scaleOrdinal()
        .range(["#1b9e77", "#d95f02", "#7570b3"])
        .domain(keys);

    g.append("g")
        .attr("transform", "translate(0," + height + ")")
        .call(d3.axisBottom(x))
        .select(".domain")
        .remove();

    g.append("g")
        .call(d3.axisLeft(y))
        .append("text")
        .attr("fill", "#000")
        .attr("transform", "rotate(-90)")
        .attr("y", 6)
        .attr("dy", "0.71em")
        .attr("text-anchor", "end")
        .text("Quantidade de pessoas");
        
    var legend = g.append("g")
        .attr("font-family", "sans-serif")
        .attr("font-size", 10)
        .attr("text-anchor", "end")
        .selectAll("g")
        .data(keys.slice().reverse())
        .enter().append("g")
        .attr("transform", function(d, i) { return "translate(0," + i * 20 + ")"; });
    
    legend.append("rect")
        .attr("x", width - 19)
        .attr("width", 19)
        .attr("height", 19)
        .attr("fill", z);
    
    legend.append("text")
        .attr("x", width - 24)
        .attr("y", 9.5)
        .attr("dy", "0.32em")
        .text(function(d) { return d; });
}

function desenhaLinha(dataPlace, data, scale, color) {
    var svg = d3.select("#vis1"),
        margin = {top: 20, right: 20, bottom: 30, left: 50},
        width = +svg.attr("width") - margin.left - margin.right,
        height = +svg.attr("height") - margin.top - margin.bottom,
        g = svg.append("g").attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    var parseTime = d3.timeParse("%H:%M");

    var x = d3.scaleTime()
        .rangeRound([0, width]);

    var y = d3.scaleLinear()
        .rangeRound([height, 0]);

    var line = d3.line()
        .x(function(d) { return x(parseTime(d.key)); })
        .y(function(d) { return y(d.value); });
        
    x.domain(d3.extent(data, function(d) { return parseTime(d.key); }));
    y.domain(d3.extent(scale, function(d) { return d; }));

    g.append("path")
        .datum(dataPlace)
        .attr("fill", "none")
        .attr("stroke", color)
        .attr("stroke-linejoin", "round")
        .attr("stroke-linecap", "round")
        .attr("stroke-width", 2)
        .attr("d", line)
        .on("mouseover", mouseover)
        .on("mouseout", function(d) {
            d3.selectAll("path")
              .style("opacity", 1);
        })

    function mouseover(d) {
        var me = this;
        d3.selectAll("path").style("opacity", function() {
            if (this === me) {
                return 1;
            } else {
                return 0.4;
            }
        }) 
    }
};

function filtraDados(dados, tempoInicial, tempoFinal) {
    var data = d3.nest()
        .key(d => d.horario_inicial)
        .rollup(v => d3.mean(v, d => parseInt(d.total_motorizados) + parseInt(d.total_ciclistas) + parseInt(d.total_pedestres)))
        .entries(dados
                .filter(d => d.horario_inicial >= tempoInicial)
                .filter(d => d.horario_inicial <= tempoFinal));

    var dataBobs = d3.nest()
        .key(d => d.horario_inicial)
        .rollup(v => d3.mean(v, d => parseInt(d.total_motorizados) + parseInt(d.total_ciclistas) + parseInt(d.total_pedestres)))
        .entries(dados.filter(d => d.local === "bobs")
                .filter(d => d.horario_inicial >= tempoInicial)
                .filter(d => d.horario_inicial <= tempoFinal));

    var dataJackson = d3.nest()
        .key(d => d.horario_inicial)
        .rollup(v => d3.mean(v, d => parseInt(d.total_motorizados) + parseInt(d.total_ciclistas) + parseInt(d.total_pedestres)))
        .entries(dados.filter(d => d.local === "jackson")
                .filter(d => d.horario_inicial >= tempoInicial)
                .filter(d => d.horario_inicial <= tempoFinal));

    var dataBurrinhos = d3.nest()
        .key(d => d.horario_inicial)
        .rollup(v => d3.mean(v, d => parseInt(d.total_motorizados) + parseInt(d.total_ciclistas) + parseInt(d.total_pedestres)))
        .entries(dados.filter(d => d.local === "burrinhos")
                .filter(d => d.horario_inicial >= tempoInicial)
                .filter(d => d.horario_inicial <= tempoFinal));

    return {"data": data, "bobs": dataBobs, "jackson": dataJackson, "burrinhos": dataBurrinhos};
}

function desenhaGrafico(dados, tempoInicial, tempoFinal) {
    var dadosFiltrados = filtraDados(dados, tempoInicial, tempoFinal);
    var data = dadosFiltrados["data"];
    var dataBobs = dadosFiltrados["bobs"];
    var dataJackson = dadosFiltrados["jackson"];
    var dataBurrinhos = dadosFiltrados["burrinhos"];

    var max = Math.max(d3.max(dataBobs, function(d) {return d.value}), d3.max(dataJackson, function(d) {return d.value}), d3.max(dataBurrinhos, function(d) {return d.value}));
    var min = Math.min(d3.min(dataBobs, function(d) {return d.value}), d3.min(dataJackson, function(d) {return d.value}), d3.min(dataBurrinhos, function(d) {return d.value}));
    var scale = [min, max];
    
    desenhaEixo(data, scale);
    desenhaLinha(dataBobs, data, scale, "#1b9e77");
    desenhaLinha(dataJackson, data, scale, "#d95f02");
    desenhaLinha(dataBurrinhos, data, scale, "#7570b3");
}

function update(dados) {
    var tempoInicial = d3.select("#dropdown1")
                         .select("select")
                         .property("value");

    var tempoFinal = d3.select("#dropdown2")
                       .select("select")
                       .property("value");

    if (tempoInicial < tempoFinal) {
        d3.select("#vis1").remove();
        d3.select("#container1").append("svg")
            .attr("id", "vis1")
            .attr("height", "500")
            .attr("width", "800");
        desenhaGrafico(dados, tempoInicial, tempoFinal);
    }
}

d3.csv('https://raw.githubusercontent.com/luizaugustomm/pessoas-no-acude/master/dados/processados/dados.csv', function(dados) {
    desenhaGrafico(dados, "06:00", "21:00");

    var dropdown1 = d3.select("#dropdown1")
        .append("select")
        .attr("id", "tempoInicial")
        .selectAll("option")
            .data(dados.filter(d => d.local === "bobs"))
            .enter()
            .append("option")
            .attr("value", function(d) {return d.horario_inicial})
            .text(function(d) {return d.horario_inicial});
    
    var dropdown2 = d3.select("#dropdown2")
        .append("select")
        .attr("id", "tempoInicial")
        .selectAll("option")
            .data(dados.filter(d => d.local === "bobs"))
            .enter()
            .append("option")
            .attr("value", function(d) {return d.horario_inicial})
            .text(function(d) {return d.horario_inicial});

    d3.select("#botao").on("click", function(d) {update(dados)});
});
</script>

<!-- vis 2 -->

<script>
var margin = {top: 20, right: 20, bottom: 30, left: 50},
    width =  800 - margin.left - margin.right,
    height = 500 - margin.top - margin.bottom;

var parseDate = d3.timeParse("%H:%M");

var svg2 = d3.select("#vis2")
    .append("g")
    .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

var x = d3.scaleTime()
    .range([0, width]);

var y = d3.scaleLinear()
    .range([height, 0]);

var line = d3.area()
    .curve(d3.curveBasis)
    .x(function(d) { return x(d.horario_inicial); })
    .y(function(d) { return y(d.total_motorizados); });

var area = d3.area()
    .curve(d3.curveBasis)
    .x(function(d) { return x(d.horario_inicial); })
    .y1(function(d) { return y(d.total_motorizados); });


d3.csv("https://raw.githubusercontent.com/luizaugustomm/pessoas-no-acude/master/dados/processados/dados.csv", function(error, data) {
  if (error) throw error;

  data.forEach(function(d) {
    d.horario_inicial = parseDate(d.horario_inicial);
    d.total_motorizados = +d.total_motorizados;
    d.total_ciclistas = +d.total_ciclistas;
    d.total_pedestres = +d.total_pedestres;
  });

  x.domain(d3.extent(data, function(d) { return d.horario_inicial; }));

  y.domain([
    d3.min(data, function(d) { return Math.min(d.total_motorizados, d.total_ciclistas + d.total_pedestres) }),
    d3.max(data, function(d) { return Math.max(d.total_motorizados, d.total_ciclistas + d.total_pedestres) })
  ]);

  svg2.datum(data);

  svg2.append("clipPath")
      .attr("id", "clip-below")
    .append("path")
      .attr("d", area.y0(height));

  svg2.append("clipPath")
      .attr("id", "clip-above")
    .append("path")
      .attr("d", area.y0(0));

  svg2.append("path")
      .attr("clip-path", "url(#clip-above)")
      .attr("d", area.y0(function(d) { return y(d.total_ciclistas + d.total_pedestres); }))
      .attr("class", "area above")
      .style("fill", "#a6cee3");

  svg2.append("path")
      .attr("clip-path", "url(#clip-below)")
      .attr("d", area)
      .attr("class", "area below")
      .style("fill", "#b2df8a");

  svg2.append("path")
      .attr("class", "line")
      .attr("d", line)
      .style("fill", "none")
      .style("stroke", "#000")
      .style("stroke-width", "1.5px");

  svg2.append("g")
      .attr("class", "x axis")
      .attr("transform", "translate(0," + height + ")")
      .call(d3.axisBottom(x));

  svg2.append("g")
      .attr("class", "y axis")
      .call(d3.axisLeft(y))
    .append("text")
      .attr("x", 2)
      .attr("y", y(y.ticks().pop()) + 0.5)
      .attr("dy", "0.32em")
      .attr("fill", "#000")
      .attr("font-weight", "bold")
      .attr("text-anchor", "start")
      .text("Quantidade de pessoas");;
});

</script>

<!-- vis 3 -->
<script>

var svg = d3.select("#vis3"),
    margin = {top: 20, right: 20, bottom: 30, left: 40},
    width = +svg.attr("width") - margin.left - margin.right,
    height = +svg.attr("height") - margin.top - margin.bottom,
    g = svg.append("g").attr("transform", "translate(" + margin.left + "," + margin.top + ")");

var parseTime = d3.timeParse("%H:%M");

var x = d3.scaleTime()
    .rangeRound([0, width]);

var y = d3.scaleLinear()
    .rangeRound([height, 0]);

var z = d3.scaleOrdinal()
    .range(["#b3cde3", "#fbb4ae"]);

d3.csv("https://raw.githubusercontent.com/luizaugustomm/pessoas-no-acude/master/dados/processados/dados.csv", function(d, i, columns) {
  d.homens_ciclistas = +d.homens_ciclistas;
  d.homens_pedestres = +d.homens_pedestres;
  d.mulheres_ciclistas = +d.mulheres_ciclistas;
  d.mulheres_pedestres = +d.mulheres_pedestres;
  d["Total de Homens"] = d.homens_ciclistas + d.homens_pedestres;
  d["Total de Mulheres"] = d.mulheres_ciclistas + d.mulheres_pedestres;
  return d;
}, function(error, data) {
  if (error) throw error;

  var keys = ["Total de Homens", "Total de Mulheres"];


  var homens = d3.nest()
       .key(d => d.horario_inicial)
       .rollup(v => d3.sum(v, d => d["Total de Homens"]))
       .entries(data);

  var mulheres = d3.nest()
       .key(d => d.horario_inicial)
       .rollup(v => d3.sum(v, d => d["Total de Mulheres"]))
       .entries(data);

   for (var i = 0; i < homens.length; i++) {
       homens[i]["Total de Homens"] = homens[i].value;
       homens[i]["Total de Mulheres"] = mulheres[i].value;
   }

  x.domain(d3.extent(data, function(d) { return parseTime(d.horario_inicial); }));
  y.domain([0, d3.max(homens, function(d) { return d["Total de Homens"] + d["Total de Mulheres"]; })]).nice();
  z.domain(keys);

  g.append("g")
    .selectAll("g")
    .data(d3.stack().keys(keys)(homens))
    .enter().append("g")
      .attr("fill", function(d) { return z(d.key); })
    .selectAll("rect")
    .data(function(d) { return d; })
    .enter().append("rect")
      .attr("x", function(d, i) { return (width / homens.length) * i; })
      .attr("y", function(d) { return y(d[1]); })
      .attr("height", function(d) { return y(d[0]) - y(d[1]); })
      .attr("width", function(d) { return width / homens.length - 1});

  g.append("g")
      .attr("class", "axis")
      .attr("transform", "translate(0," + height + ")")
      .call(d3.axisBottom(x));

  g.append("g")
      .attr("class", "axis")
      .call(d3.axisLeft(y).ticks(null, "s"))
    .append("text")
      .attr("x", 2)
      .attr("y", y(y.ticks().pop()) + 0.5)
      .attr("dy", "0.32em")
      .attr("fill", "#000")
      .attr("font-weight", "bold")
      .attr("text-anchor", "start")
      .text("Número de não motorizados");

  var legend = g.append("g")
      .attr("font-family", "sans-serif")
      .attr("font-size", 10)
      .attr("text-anchor", "end")
    .selectAll("g")
    .data(keys.slice().reverse())
    .enter().append("g")
      .attr("transform", function(d, i) { return "translate(0," + i * 20 + ")"; });

  legend.append("rect")
      .attr("x", width - 19)
      .attr("width", 19)
      .attr("height", 19)
      .attr("fill", z);

  legend.append("text")
      .attr("x", width - 24)
      .attr("y", 9.5)
      .attr("dy", "0.32em")
      .text(function(d) { return d; });
});

</script>
