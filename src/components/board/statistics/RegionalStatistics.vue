<template>
<div>
    <side-bar></side-bar>
    <div id="content">
      <div class="title-wrapper">
          <span class="title">객체종류 지역별 통계</span>
      </div>
      <div class="select-condition">
        <table class="condition-table">
              <colgroup>
                <col width="15%">
              </colgroup>
              <tbody>
                <tr class="condition-table-body">
                  <td>검색기간</td>
                  <td>
                    <input type="date" v-model="firstDate"/><input type="time" v-model="firstTime"/> ~
                    <input type="date" v-model="lastDate"/><input type="time" v-model="lastTime"/>
                    <!-- {{firstDate + " " + firstTime}}
                    {{lastDate + " " + lastTime}} -->
                  </td>
                </tr>
                <tr class="condition-table-body">
                  <td>객체 종류</td>
                  <td>
                    <select name="selectingObject" v-model="selectObject" >
                      <option v-for="(object, index) in objectCategory" :key="index">
                          {{object}}
                      </option>
                    </select>
                    <button class="button-add" v-on:click="addObject(selectObject)">추가</button>
                  </td>
                </tr>
                <tr class="condition-table-body">
                  <td></td>
                  <td>
                     <span class="groupbutton" v-for="(object,index) in objectArr" :key="index+1">
                                  {{object}}
                    <span class="groupremove" type="button" v-on:click="removeObject(index+1)">
                        <i class="closeBtn fas fa-times"></i>
                    </span>
                </span>
                  </td>
                </tr>
                <tr class="condition-table-body">
                  <td>검색 지역</td>
                  <td>
                    <select name="selectingArea1" v-model="selectArea1">
                      <option value="경상북도">경상북도</option>
                    </select>
                    <select name="selectingArea2" v-model="selectArea2">
                        <option value="고령군">고령군</option>
                    </select>
                    <button class="button-add" v-on:click="addArea(selectArea1, selectArea2)">추가</button>
                  </td>
                </tr>
                <tr class="condition-table-body">
                  <td></td>
                  <td>
                    <span  class="groupbutton" v-for="(area,index) in areaArr" :key="index">
                        {{area}}
                        <span class="groupremove" type="button" v-on:click="removeArea(index)">
                            <i class="closeBtn fas fa-times"></i>
                        </span>
                    </span>
                  </td>
                  </tr>
              </tbody>
        </table>
        <button class="button-cancle" v-on:click="makeExcelFile()">내보내기</button>
        <button class="button-add" v-on:click="searchObjectArea(firstDate,firstTime,lastDate,lastTime)">조회</button>
        <div class="content-result">
              <table class="table1">
                <colgroup>
                    <col width="200px">
                    <col width="200px">
                    <col width="200px">
                    <col width="150px">
                    <col width="600px">
                </colgroup>
                  <thead>
                      <tr class="table1-head-title">
                          <th>지역</th>
                          <th>차</th>
                          <th>사람</th>
                          <th>합계</th>
                      </tr>
                  </thead>
                  <tbody>
                      <tr class="table1-body" v-for="(data,index) in searchData" :key="index">
                          <td>{{data.area}}</td>
                        <td>{{data.car}}({{data.carPercent}}%)</td>
                        <td>{{data.person}}({{data.personPercent}}%)</td>
                        <td>{{data.sum}}</td>
                      </tr>
                  </tbody>
              </table>
                 <table class="secondTable">
                    <thead>
                        <th>차트</th>
                    </thead>
                    <tbody>
                         <tr class="tBody2">        
                             <td v-if='showChart'><pie-chart :data="chartData" :options="chartOptions"></pie-chart></td>
                      </tr>
                      </tbody>
        </table>
        </div>
      </div>
    </div>
</div>
</template>
<script>
import Xlsx from 'xlsx'
import PieChart from './PieChart'
import SideBar from '../../common/SideBar.vue'

export default {
    components:{
        PieChart,
        SideBar
    },
    data(){
        return{
            getObjectAreaData:[],
            objectCategory:["차","사람"],
            selectObject:'',
            selectArea1:'',
            selectArea2:'',
            selectArea:'',
            objectArr:[],
            areaArr:[],
            firstDate:'',
            firstTime:'',
            lastDate:'',
            lastTime:'',

            //테이블 출력 데이터 모아논 배열
            searchData:[],

            //차트에 보여줄 데이터 0번이 차 1번이 사람
            chartArr:[],

            //vue-chart.js
            showChart:false,
                        //vue-chart.js
            chartData:{
               show: false,
               hoverBackgroundColor: "red",
               hoverBorderWidth: 10,
               labels: ["차", "사람"],
               datasets: [
                   {
                       label: "처리현황",
                       backgroundColor: ["#6DA0F1", "#467FD3"],
                       data: [],
                   }
               ]
           },
           chartOptions: {
               title:{
                   display:true,
                   text: '객체종류',
                   position: "bottom"
               },
                responsive: true,
                legend: {
                    position: 'right',
                    },
                maintainAspectRatio: true,
               pieceLabel: {
                    mode: 'percentage',
                    precision: '0'
                }
           },
        }
    },
    methods:{
        getObjectAreaToJson(){
            this.$http.get('http://localhost:3000/result_of_area')
            .then((res) => {
                console.log('getdata: ', res.data)
                this.getObjectAreaData = res.data
            })
        },
        isExistObject(selectObject){
            let returnFlag = false;
            for(let i in this.objectArr){
                if(this.objectArr[i] == selectObject){
                    returnFlag = true;
                }
            }
            console.log(returnFlag);
            return returnFlag;
        },
        isExistArea(selectArea){
            let returnFlag = false;
            for(let i in this.areaArr){
                if(this.areaArr[i] == selectArea){
                    returnFlag = true;
                }
            }
            return returnFlag;
        },
        addObject(selectObject){
            if(!this.isExistObject(selectObject)){
                this.objectArr.push(selectObject)
            }else{
                alert("이미 선택한 객체 입니다.")
            }
        },
        addArea(selectArea1, selectArea2){
            if(!this.isExistArea(selectArea1 + selectArea2)){
                this.areaArr.push(selectArea1 + " "+ selectArea2)
            }else{
                alert("이미 선택한 지역 입니다")
            }
        },
        removeObject(index){
            this.objectArr.splice(index-1,1);
        },
        removeArea(index){
            this.areaArr.splice(index,1);
        },
        searchObjectArea(firstDate,firstTime,lastDate,lastTime){
            this.searchData.splice(0)
            this.chartArr = [];  // 차트에 들어가는 수 초기화 // 추가해줌!!
            let fDateTime = firstDate+" "+firstTime
            let lDateTime = lastDate+" "+lastTime

            if(!firstDate&&!firstTime&&!lastDate&&!lastTime){
                alert("기간을 입력하세요")
            }else{
                for(let i=0; i<this.areaArr.length; i++){
                    this.searchData.push({
                        car : 0,
                        carPercent:0,
                        person : 0,
                        personPercent:0,
                        flame : 0,
                        smoke : 0,
                        sum : 0,
                        area : this.areaArr[i]
                    })
                }
                for(let i=0; i<this.searchData.length; i++){
                    for(let j=0; j<this.getObjectAreaData.length; j++){
                        if(this.isbetweenDate(fDateTime,lDateTime,this.getObjectAreaData[j].created_at)
                        && this.areaArr[i]==this.getObjectAreaData[j].area1+" "+this.getObjectAreaData[j].area2){
                            for(let k=0; k<this.objectArr.length; k++){
                                if((this.objectArr.length>0) && (this.objectArr[k]=="차") && (this.getObjectAreaData[j].car == "o")){
                                    this.searchData[i].car++;
                                    this.searchData[i].sum++;
                                }
                                if((this.objectArr.length>0) && (this.objectArr[k]=="사람") && this.getObjectAreaData[j].person == "o"){
                                    this.searchData[i].person++;
                                    this.searchData[i].sum++;
                                }
                                if((this.objectArr.length>0) && (this.objectArr[k]=="화제") && this.getObjectAreaData[j].flame == "o"){
                                    this.searchData[i].flame++;
                                    this.searchData[i].sum++;
                                }
                                if((this.objectArr.length>0) && (this.objectArr[k]=="연기") && this.getObjectAreaData[j].smoke == "o"){
                                    this.searchData[i].smoke++;
                                    this.searchData[i].sum++;
                                }
                            }
                            this.searchData[i].carPercent = (this.searchData[i].car/this.searchData[i].sum*100).toFixed(2)
                            this.searchData[i].personPercent = (this.searchData[i].person/this.searchData[i].sum*100).toFixed(2)
                        }
                    }
                }

            }

            // this.chartArr = [];  // 차트에 들어가는 수 초기화 // 추가해줌!!
            // 차트를 위한 데이터 입력
            for( let i=0; i<this.searchData.length; i++){
                this.chartArr.push({
                    car: this.searchData[i].car,
                    person: this.searchData[i].person
                })
            }
            this.chartDataPush()
        },
        isbetweenDate(fDateTime,lDateTime,searchDate){
            let returnFlag=false
            const moment = require('moment');
            if(moment(searchDate).isBetween(fDateTime, lDateTime, undefined, '()')){
                returnFlag = true
            }
            // console.log(returnFlag)
            return returnFlag
        },

        // 엑셀
        makeExcelFile(){
            console.log("엑셀내보내기");
            const workBook = Xlsx.utils.book_new();
            const workSheet = Xlsx.utils.json_to_sheet(this.searchData);
            Xlsx.utils.book_append_sheet(workBook, workSheet, '시트이름');
            Xlsx.writeFile(workBook, '파일이름.xlsx')
        },

        chartDataPush(){
               //배열 초기화
            this.showChart = true;
            var num1 = 0;
            var num2 = 0;
            num1 = this.chartArr[0].car
            num2 = this.chartArr[0].person
            console.log("car"+num1)
            console.log("넘넘넘넘");
            console.log("person"+num2);
            this.chartData.datasets[0].data = [];
            console.log(this.chartData.datasets[0]);
            this.chartData.datasets[0].data.push(num1)
            this.chartData.datasets[0].data.push(num2)
        }

    },
    beforeMount(){

    },
    mounted() {
        this.getObjectAreaToJson()

    },
    beforeUpdate(){ //데이터 변경되면 가상돔을 화면을 다시 그리기 전에 호출

    },
    updated(){ // 화면이 그려고 나면 호출

    },
    beforeDestroy(){

    }

}
</script>
<style lang="">
.table1{
    width:1300px;
    float:left;
    
}
.secondTable{
    
    width:600px;
    border-left: 1px;
    border-bottom: 1px solid;
    text-align:center;
    vertical-align: middle;
    height: 40px;
    background-color: #f6f7fb;
}
 .table1-head-title-status {
    text-align:center;
    vertical-align: middle;
    height: 40px;
    background-color: #f6f7fb;
  }
</style>
