---
title: "Incluindo gráficos em um PDF usando Laravel e Chartjs"
date: 2020-04-02
---

Se você teve que gerar algum pdf que precisasse utilizar gráficos, sabe o quão difícil é conseguir incluir um gráfico dentro de um pdf, principalmente se você estiver usando gráficos em Javascript.

## Referências
[Quickchart.io](https://quickchart.io/)
[Chartjs](https://www.chartjs.org/)
[Laravel](https://laravel.com/)

## Quickchart.io
Depois de procurar várias formas para incluir os gráficos dentro de um pdf, acabei encontrando um site que se chama [quickchart.io](https://quickchart.io/), o que ele faz é basicamente transformar gráficos javascript em imagens estáticas que podem ser utilizadas em qualquer lugar, inclusive em um pdf.

Ele funciona da seguinte maneira:
```
https://quickchart.io/chart?c={type:'bar',data:{labels:[2012,2013,2014,2015,2016],datasets:[{label:'Users',data:[120,60,50,180,120]}]}}
```
Você envia um `query param` na url do site com o nome `c` e o valor é um objeto com as configurações e dados do gráfico, da mesma forma que você cria no `chartjs`

O resultado dessa requisição é uma imagem png com o gráfico:

![Imagem do gráfico](https://quickchart.io/chart?c={type:%27bar%27,data:{labels:[2012,2013,2014,2015,2016],datasets:[{label:%27Users%27,data:[120,60,50,180,120]}]}})


## Incluindo a imagem no PDF usando Laravel
Nesse caso, estarei usando o [dompdf](https://github.com/dompdf/dompdf) e o `Laravel 5.5`.

Primeiro passo, tenha os dados na estrutura correta que pede o `chartjs`.
```php
$chartData = [
  "type" => 'horizontalBar',
    "data" => [
      "labels" => ['Coluna 1', 'Coluna 2', 'Coluna 3'],
        "datasets" => [
          [
            "label" => "Dados", 
            "data" => [100, 60, 20],
            "backgroundColor" => ['#27ae60', '#f1c40f', '#e74c3c']
          ], 
        ],
      ]
  ];     
```

Depois, transforme essa estrutura em `json`
```php
$chartData = json_encode($chartData);
```

e mande ela pra URL do `quickchart.io` usando `urlencode`
```php
$chartURL = "https://quickchart.io/chart?width=300&height=200&c=".urlencode($chartData);
```
Sim, é possível você escolher o `width` e o `height` do gráfico. A api fornece alguns parâmetros personalizáveis, é só dar uma olhadinha na documentação do [quickchart.io](https://quickchart.io/) 🥰

Feito isso, se você colocar a variável `$chartURL` dentro de uma tag `<img>`, a imagem deve aparecer. Porém, eu tive alguns problemas com inserir a variável direto e a imagem não aparecer dentro do pdf, então eu prefiro transformar essa imagem em `base64` e ai sim enviar para o pdf.

Para transformar a imagem em `base64` é simples:
```php
$chartData = file_get_contents($chartURL); 
$chart = 'data:image/png;base64, '.base64_encode($chartData);
```

Após, isso a variável `$chart` já vai possuir o `base64` da imagem, como vocês podem notar, eu concatenei no começo da variável o `data:image/png;base64,'` porque é algo necessário para a tag `<img>` entender que a fonte da imagem é `base64`.

Após todos esses passos, estamos prontos para enviar essa variável para o template `blade` que irá gerar o pdf e incluir o gráfico dentro dele.

Para enviar e incluir a imagem é simples, basta fazer isso:

```php
$pdf = PDF::loadView('report.pdf', ['chart' => $chart]);
return $pdf->stream();
```

E no arquivo `report.blade.php` (template do `report.pdf`)

```php
<img src="{{$chart}}">
```
 
Dessa maneira, o gráfico aparecerá corretamente dentro do seu pdf.

Qualquer dúvida, fico a disposição para ajudar.

Meu github
https://github.com/leotheodoro

