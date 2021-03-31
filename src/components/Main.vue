<template>
  <div id="main" class="py-4">
    
    <div class="jumbotron container">
      <h1 class="display-4 text-center">TypeQuickie</h1>
      <p class="lead">Welcome, Free online typing application to see how fast you type! 
        Includes bunches of text alternatives and many test lengths. 
        Simple and fun approach to test and improve your typing speed.</p>
      <h5 v-if="!finished" class="d-flex justify-content-between">
        <div>
          Correct: {{correctCount}} - Wrong: {{wrongCount}}
        </div>
        <div class="form-row align-items-center">
          <!-- <div class="col-md-3"> -->
            <div class="input-group text-primary">
              <label for="timer">Timer: </label>
              <select id="timer" :disabled="isRunning" class="custom-select ml-3" v-model="selectedTimer">
                <option value="15">15s</option>
                <option value="30" selected>30s</option>
                <option value="60">60s</option>
              </select> 
            </div>
        </div>
      </h5>
      <hr class="my-4">

      <div v-if="finished" class="alert alert-light">
        <h4>Finished</h4>
        <p class="text-info">Correct words percentage: {{correctPercentage}}%</p>
        <p class="text-dark">Total words: {{wordsWritten}}</p>
        <p class="text-success">Correct words: {{correctCount}}</p>
        <p class="text-danger">Wrong words: {{wrongCount}}</p>
        <p class="text-primary">RECORD: {{prevRecord}}</p>
        <hr>
        <button @click="start" class="btn btn-success">Play again</button>
      </div>

      <template v-else>
        <div class="card">
          <div class="card-body">
            <span v-for="(word, index) in words.filter((data, index)=>{
                return index < 20;
              })" :key="index" 
              class="mr-2 word" 
              :class="{
                'current-word':index==0,
                'bg-danger':index ==0 && !isCorrect,
              }"
            >
              {{word}}
            </span>
          </div>
        </div>
        <div class="card card-dark bg-dark my-4">
          <div class="card-body">
            <div class="input-group input-group-lg">
              <input type="text" class="form-control" v-model="writingWord">
              <button class="btn btn-light disabled" disabled>{{timer}}s</button>
              <button :disabled="isRunning" class="btn btn-primary" type="button" @click="getWords">Reset</button>
            </div>
          </div>
        </div>
        <p class="text-center">
          <button :disabled="!isRunning" class="btn btn-info" type="button" @click="start" :class="{'button-disabled':!isRunning}">Start Again</button>
        </p>
      </template>
      <p class="text-center mt-5 profile">
        <a class="text-dark text-decoration-none mr-3" target="_blank" href="http://turalhasanovportfolio.herokuapp.com/">
          <i class="bi bi-globe"></i> Portfel
        </a>
        <a class="text-dark text-decoration-none" target="_blank" href="https://github.com/TuralHasanov11/">
          <i class="bi bi-github"></i>
          Github
        </a>
      </p>
    </div>
    
  </div>
</template>

<script>
import wordsList from '@/assets/words.json';

export default {
  name: 'Main',
  data(){
    return {
      wordsList: wordsList,
      words:[],
      writingWord:null,
      isCorrect:true,
      correctCount:0,
      wrongCount:0,
      timer:null,
      selectedTimer:localStorage.getItem('timer')?localStorage.getItem('timer'):30,
      interval:false,
      isRunning:false,
      finished:false,
      isLoading: true,
      prevRecord:null,
      prevCorrectPercentage:null,
    }
  },

  computed:{
    wordsWritten(){
      return 300 - this.words.length;
    },

    correctPercentage(){
      return this.correctCount ? Number(this.correctCount/this.wordsWritten*100).toFixed(2) : 0;
    }
  },

  watch:{
    writingWord(val){
      if(!val || val==' '){
        this.writingWord='';
        return;
      }
      if(!this.isRunning){
        this.toggleTimer();
      }

      let word = this.words[0].slice(0,val.length).toUpperCase();
      let input = val.replace(' ','').toUpperCase();
      this.isCorrect = word === input;

      if(val.indexOf(' ') !== -1){
        this.isCorrect ? this.correctCount++ : this.wrongCount++;
        this.words.splice(0, 1);
        this.writingWord = '';
        this.isCorrect = true;
      }
    },

    selectedTimer(value){
      this.timer = value;
    }
  },

  mounted(){
    this.timer = this.selectedTimer;
    this.getWords();
    
  },

  methods:{

    start(){
      clearInterval(this.interval);
      this.words = [];
      this.finished=false;
      this.timer = localStorage.getItem('timer');
      this.word = '';
      this.writingWord = null;
      this.interval = false;
      this.isCorrect = true;
      this.isRunning = false;
      this.correctCount = 0;
      this.wrongCount = 0;
      this.getWords();
      this.isLoading = false;
    },

    getWords(){
      this.shuffleArray();
      this.words = this.wordsList.splice(0, 300);
    },

    toggleTimer(){
      this.isRunning = true;
      this.timer = this.selectedTimer;
      localStorage.setItem('timer', this.selectedTimer);
      this.interval = setInterval(this.timing, 1000);
    },

    timing(){
        if(this.timer === 0){
            
          let record = this.setRecord();
          this.prevRecord = record.prevRecord;
          this.prevCorrectPercentage = record.prevCorrectPercentage;

          clearInterval(this.interval);
          this.finished = true;
          return;
        }
         this.timer--;
    },

    setRecord(){
      let record = {correctCount:this.correctCount, correctPercentage:this.correctPercentage};
      let prev = {};
      switch (localStorage.getItem('timer')) {
        case '15':
          if(localStorage.getItem('record_15')==null || (this.correctCount > localStorage.getItem('record_15') || (this.correctCount == localStorage.getItem('record_15') && this.correctPercentage > localStorage.getItem('correct_percentage_15') ))){
            localStorage.setItem('record_15', record.correctCount);
            localStorage.setItem('current_correct_percentage_15', record.correctPercentage);
            prev = {prevRecord:record.correctCount, prevCorrectPercentage:this.correctPercentage};
          }
          break;
        case '30':
          if(localStorage.getItem('record_30')==null || (this.correctCount > localStorage.getItem('record_30') || (this.correctCount == localStorage.getItem('record_30') && this.correctPercentage > localStorage.getItem('correct_percentage_30') ))){
            localStorage.setItem('record_30', record.correctCount);
            localStorage.setItem('current_correct_percentage_30', record.correctPercentage);
            prev = {prevRecord:record.correctCount, prevCorrectPercentage:this.correctPercentage};
          }
          
          break;
        case '60':
          if(localStorage.getItem('record_60')==null || (this.correctCount > localStorage.getItem('record_60') || (this.correctCount == localStorage.getItem('record_60') && this.correctPercentage > localStorage.getItem('correct_percentage_60') ))){
            localStorage.setItem('record_60', record.correctCount);
            localStorage.setItem('current_correct_percentage_60', record.correctPercentage);
            prev = {prevRecord:record.correctCount, prevCorrectPercentage:this.correctPercentage};
          }
          break;
    
        default:
        break;
      }
      return prev;
    },

    shuffleArray() {
      for (let i = this.wordsList.length - 1; i > 0; i--) {
          const j = Math.floor(Math.random() * (i + 1));
          [this.wordsList[i], this.wordsList[j]] = [this.wordsList[j], this.wordsList[i]];
      }
    }
  }
}
</script>

<style scoped>
  .word{
    font-size: 1.5rem;
    font-weight: 400;
  }

  .current-word{
    background-color: #ddd;
    padding: 0.15rem;  
    border-radius: 0.15rem;
  }

  .profile{
    font-size: 1.2rem !important;
  }

  .button-disabled{
    cursor: not-allowed;
  }
</style>