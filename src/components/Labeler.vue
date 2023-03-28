<template>
    <div>
        <div v-if="!klines.length" style="display: flex; flex-direction: row; align-items: center; justify-content: center; width: 100%; margin-bottom: 1rem; padding-top: 0.5rem;">

            <div>
            <label class="custom-file-upload">
                    New File
                    <input type="file" ref="doc" @change="readFile()" />
                </label>
            </div>

            <div style="margin-left: 1rem">
                <label class="custom-file-upload">
                    Load File
                    <input @change="load()" ref="lod" type="file" />
                </label>
            </div>
            
        </div>
        

        <div v-if="klines.length" style="margin-bottom: 1rem; padding-top: 1.3rem;">
            <label>
                    {{ file.name }}
                </label>
            </div>

        <div v-if="chart">
                <trading-vue
                :data="chart" 
                :colorBack="'#1D1E20'"
                :colorGrid="'#27272B'"
                :colorCandleUp="'#2BAE66'"
                :colorWickUp="'#2BAE66'"
                :colorCandleDw="'#F02D3A'"
                :colorWickDw="'#F02D3A'"
                :colorTitle="'#FCF6F5'"
                :colorText="'#FCF6F5'"
                :width="width"
                :height="height"
                :titleTxt="''"
                :indexBased="true"
                ref="tradingVue"></trading-vue>            
        </div>

        <div v-if="loading" style="position: absolute; top:0;left:0;right:0;bottom:0; background: '#eeeeee'">
            
        </div>

        <div style="margin-top: 0.5rem" v-if="klines.length && breakPoints.length">
            <button @click="save()">Save</button>
        </div>

    </div>
</template>


<script type="text/javascript" src="https://canvasjs.com/assets/script/jquery-1.11.1.min.js"></script>
<script type="text/javascript" src="https://canvasjs.com/assets/script/canvasjs.stock.min.js"></script>

<script>
  import TradingVue from 'trading-vue-js'
  import DataCube from 'trading-vue-js/src/helpers/datacube'
   
  export default {
    name: 'Labeler',
    components: {TradingVue},
    data() {
        return {
            file: null,
            height: window.innerHeight - 64 - 32 - 16,
            width: window.innerWidth - 16,
            klines: [],
            done: false,
            chart: new DataCube({ ohlcv: [], onchart: [], offchart: [] }),
            loading: false,
            breakPoints: [],
            spliters: {
                    "name": "spliters",
                    "type": "Splitters",
                    "data": [],
                    "settings": {
                        "lineColor": "#FCF6F5",
                        "legend": false,
                    }
            },
            csvSplitBy: ","
        }
    },
    mounted(){
        document.body.onkeyup = (e) => {
            if(e.code == 'ArrowUp' || e.key == 'ArrowUp' || 
                e.code == 'ArrowDown' || e.key == 'ArrowDown'|| 
                e.code == 'ArrowRight' || e.key == 'ArrowRight') {
                const cursor = this.$refs.tradingVue.getCursor()
                let value = "";
                let text = "";
                if(e.code == 'ArrowUp' || e.key == 'ArrowUp'){
                    value = 1;
                    text = "Bulls" 
                }
                if(e.code == 'ArrowDown' || e.key == 'ArrowDown'){
                    value = -1;
                    text = "Bears" 
                }
                if(e.code == 'ArrowRight' || e.key == 'ArrowRight'){
                    value = 0;
                    text = "Sideways" 
                }
                this.breakPoints.push({index: cursor.i, value })
                const data = [
                            cursor.t,
                            text
                        ]
                
                this.spliters.data.push(data);
                this.chart.set("onchart.spliters", this.spliters)

            }
            if(e.code == 'Backspace' || e.key == 'Backspace') {
                this.spliters.data.pop();
                this.breakPoints.pop()
                this.chart.set("onchart.spliters", this.spliters)
            }
        }
    },
    methods: {
        readFile() {
            this.loading = true;
            this.file = this.$refs.doc.files[0];
            const reader = new FileReader();
            
            reader.onload = (res) => {
                this.loadData(res.target.result);
                console.log("reading")
            };
            reader.onerror = (err) => console.log(err);
            reader.readAsText(this.file);
        },
        loadData(res) {
            let data = res.split("\n")
            this.klines = this.prepareData(data);
            this.chart.set('chart.data', this.klines);
            this.chart.add('onchart', this.spliters)
            setTimeout(() => {
                this.$refs.tradingVue.setRange(0, 100);
                this.loading = false;
            }, 2000)
        },
        prepareData(rows, breakPoints = false) {
            let _data = [];
            let index = 0;
            let tempValue = null;
            let tempTime = null;
            for (let price of rows) {
                price = price.split(this.csvSplitBy);
                let [time, open, high, low, close, volume, closeTime, assetVolume,
                        trades, buyBaseVolume, buyAssetVolume, ignored, breakPoint] = price;

                // if(breakPoints)
                //     [time, open, high, low, close, volume, closeTime, assetVolume,
                //         trades, buyBaseVolume, buyAssetVolume, ignored] = price;
                    
                time = (new Date(time)).getTime()
                if( isNaN(open)) continue

                _data.push([
                        time,
                        parseFloat(open),
                        parseFloat(high),
                        parseFloat(low),
                        parseFloat(close),
                        parseFloat(volume)]);
                
                if(breakPoints){                    
                    if(tempValue && tempValue != breakPoint) {
                        let text = "";
                        if(tempValue == "1") text = "Bulls";
                        if(tempValue == "0") text = "Sideways";
                        if(tempValue == "-1") text = "Bears";
                        let bpData = [
                            tempTime,
                            text
                        ]
                        this.breakPoints.push({index: index - 1, value: tempValue })
                        this.spliters.data.push(bpData);
                    }

                    tempValue = breakPoint;
                    tempTime = time;
                }
                index++;
            }

            return _data;
        },
        on_button_click(event) {
            console.log(event)
        },
        load() {
            this.loading = true;
            this.file = this.$refs.lod.files[0];
            const reader = new FileReader();
            
            reader.onload = (res) => {
                let data = res.target.result.split("\n")
                this.klines = this.prepareData(data, true);
                this.chart.set('chart.data', this.klines);
                setTimeout(() => {
                    this.chart.add('onchart', this.spliters)
                    const index = this.breakPoints[this.breakPoints.length - 1].index;
                    // this.chart.set("onchart.spliters", this.spliters);
                    this.$refs.tradingVue.setRange(index, index + 100);
                    this.loading = false;
                }, 2000)
            };
            reader.onerror = (err) => console.log(err);
            reader.readAsText(this.file);

            
        },
        save() {
            // load file
            const reader = new FileReader();
            
            this.content = "check the console for file output";
            reader.onload = (res) => {
                console.log("saving...")
                let data = res.target.result.split("\n")
                let replace = true;
                // check if has label field
                if(!data[0].includes(`${this.csvSplitBy}label`)) {
                    data[0] += `${this.csvSplitBy}label`
                    replace = false;
                }
                let point = this.breakPoints.shift();
                for(let i = 1; i<(data.length - 1); i++){
                    if(i - 1 > point.index && this.breakPoints.length)
                        point = this.breakPoints.shift()
                    
                    if(replace) {
                        data[i] = data[i].split(this.csvSplitBy);
                        data[i].pop();
                        data[i] = data[i].join(this.csvSplitBy);
                    }
                    data[i] += `${this.csvSplitBy}${point.value}`
                }
                this.downloadFile(data.join("\n"))
            };
            reader.onerror = (err) => console.log(err);
            reader.readAsText(this.file);
        },
        downloadFile(text) {
            // Create element with <a> tag
            const link = document.createElement("a");

            // Create a blog object with the file content which you want to add to the file
            const file = new Blob([text], { type: 'text/csv' });

            // Add file content in the object URL
            link.href = URL.createObjectURL(file);

            // Add file name
            link.download = "LABELED-" + this.file.name;

            // Add click event to <a> tag to save file.
            link.click();
            URL.revokeObjectURL(link.href);
        }
    }
  }
  </script>
  
  <!-- Add "scoped" attribute to limit CSS to this component only -->
  <style scoped>
input[type="file"] {
    display: none;
}

.custom-file-upload {
    border: 1px solid #ccc;
    display: inline-block;
    padding: 6px 12px;
    cursor: pointer;
}
  </style>
  