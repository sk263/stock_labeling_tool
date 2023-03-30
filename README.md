# Stock Labeling Software Documentation
## Introduction
This software is designed to help users label stock charts quickly and efficiently using arrow keys. The arrow keys allow the user to label different parts of the chart as bullish, bearish, or sideways. The software allows the user to load new charts in .csv format and save labeled charts for future use.

## Usage
To use this software, simply open the application in your web browser. Once the application is loaded, you can use the arrow keys to label different parts of the chart.

- Arrow Up: Label chart as bullish (1)
- Arrow Right: Label part of chart as sideways (0)
- Arrow Down: Label part of chart as bearish (-1)


To load a new chart, click the "New Chart" button and select the .csv file containing the chart data in the format below. Once the chart is loaded, you can use the arrow keys to label it.

To save a labeled chart, click the "Save Chart" button. The labeled chart will be saved as a .csv file with an additional "Label" column that marks the direction of price.

To load a previously labeled chart, click the "Load Chart" button and select the .csv file containing the labeled chart data. The labeled chart will be loaded and you can continue labeling it.

## Chart Data Format
The chart data should be in .csv format with the following columns:

```
[
  1620921600000,  // Open time in milliseconds
  "39223.47000000",  // Open price
  "39790.01000000",  // High price
  "38968.22000000",  // Low price
  "39790.01000000",  // Close price
  "59457.41441700",  // Volume of the asset traded during the candlestick period
  1621007999999,  // Close time in milliseconds
  "2348363057.38886900",  // Quote asset volume
  2840,  // Number of trades
  "29244.61938500",  // Taker buy base asset volume
  "1157983211.79127868",  // Taker buy quote asset volume
  "0"  // Ignore
]
```

This format of data can be fetched from Binance API


The columns should be in the order listed above, and each row should represent a single candlestick/bar of the chart. For labeled charts, there should be an additional "Label" column that marks the direction of price (-1 for bearish, 0 for sideways, and 1 for bullish).

## Limitations
Please note that this software is intended for educational and demonstration purposes only. It is not intended to provide financial or investment advice, and the labeling of charts should not be taken as a reliable indicator of future market performance. Always consult a qualified financial advisor before making investment decisions.

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```
