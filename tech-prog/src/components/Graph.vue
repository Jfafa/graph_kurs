<template> 
  <div>
    
    <h1>Живучесть графа</h1>
    <div id="col-1">
      <input v-model="sid1" placeholder="sid">
      <input v-model="tid1" placeholder="tid">
      <input v-model="val1" placeholder="val">
      <button @click="setLinks1">Add link</button>
      <button @click="setNodes1">Add node</button>
      <d3-network :net-nodes="nodes" :net-links="links" :options="options">
      </d3-network>
    </div>
    
    <div id="col-2">
      <input v-model="sid2" placeholder="sid">
      <input v-model="tid2" placeholder="tid">
      <input v-model="val2" placeholder="val">
      <button @click="setLinks2">Add link</button>
      <button @click="setNodes2">Add node</button>
      <d3-network :net-nodes="nodes2" :net-links="links2" :options="options2">
      </d3-network>
    </div>
    <button @click="handleCalculation">Calculate graph</button>
  </div>
</template>

<script>
import D3Network from 'vue-d3-network'
import Djikstra from '../calculationServices/dijkstra'

export default {
  components: {
    D3Network
  },
  data () {
    return {
      sid1: '',
      tid1: '',
      val1: '',
      sid2: '',
      tid2: '',
      val2: '',
      nodes1Input: '',
      nodes2Input: '',
      nodes: [],
      links: [],
      nodes2: [],
      links2: [],
      options:
      {
        force: 2000,
        nodeSize: 10,
        nodeLabels: true,
        linkWidth:5,
        linkLabels: true,
      },
      options2:
      {
        force: 2000,
        nodeSize: 10,
        nodeLabels: true,
        linkWidth:4,
        linkLabels: true,
      },
      matrix:[],
      matrix2:[],
    }
  },
  methods: {
    getBiggestWeight(matrix){
      let biggestWeight = 0;
      for(let i = 0; i < matrix[0].length; i++){
        let weights = Djikstra.buildWeightMap(matrix, i);
        weights.forEach(element => {
          if(element != Infinity && element > biggestWeight) {
            biggestWeight = element;
          }
        });
      }
      return biggestWeight;
    },
    handleCalculation(){
      this.makeMatrix1();
      this.makeMatrix2();
      alert(this.makeFO(this.matrix))
      alert(this.makeFO(this.matrix2))
      let matrix1Result = this.getNumberOfDeletedElements(this.matrix);
      let matrix2Result = this.getNumberOfDeletedElements(this.matrix2);
      if(this.nodes.length === 2 && this.nodes2.length === 2){
        alert('Равны по признаку приведения к тривиальному графу')
      }else{
        if(matrix1Result === matrix2Result && matrix1Result !== 0) alert('Равны');
        if(matrix1Result !== matrix2Result && matrix1Result !== 0) alert('Не равны');
        if(matrix1Result === 0) alert('Для матрицы 1 невозможно вычислить. Т.К. нужно удалить больше 2х элементов');
        if(matrix2Result === 0) alert('Для матрицы 2 невозможно вычислить. Т.К. нужно удалить больше 2х элементов');
      }
    },
    getNumberOfDeletedElements(matrix){
      let biggestWeight = this.getBiggestWeight(matrix)

      for(let i = 0; i < matrix[0].length; i++){
        let mat = this.deleteMatrixElement(matrix, i)
        let be = this.getBiggestWeight(mat)
        if(be > biggestWeight) return 1;
      }

      for(let i = 0; i < matrix[0].length; i++){
        for(let k = 0; k < matrix[0].length; k++){
          if(i === k) continue;
          let mat = this.deleteMatrixElement(matrix, i);
          mat = this.deleteMatrixElement(mat, k);
          let be = this.getBiggestWeight(mat)
          if(be > biggestWeight) return 2;
        }
      }

      return 0;
    },
    deleteMatrixElement(matrix, index){
      let newMatrix = JSON.parse(JSON.stringify(matrix))
        for(let j = 0; j < newMatrix.length; j++){
          newMatrix[j][index] = 0;
          newMatrix[index][j] = 0;
        }
      return newMatrix;
    },

    setLinks1(){
      this.links.push({ sid: this.sid1, tid: this.tid1, name: this.val1 })
    },

    setLinks2(){
      this.links2.push({ sid: this.sid2, tid: this.tid2, name: this.val2 })
    },

    setNodes1(){
      this.nodes.push({id: this.nodes1Input})
    },

    setNodes2(){
      this.nodes2.push({id: this.nodes2Input})
    },

    makeMatrix1(){
      let matr = []
      for(let i = 0; i < this.nodes.length; i++) matr.push(Array(this.nodes.length).fill(0))
      this.links.forEach(link => {
        matr[+link.source.id][+link.target.id] = +link.name;
        matr[+link.target.id][+link.source.id] = +link.name;
      })
      this.matrix = matr;
    },

    makeMatrix2(){
      let matr = []
      for(let i = 0; i < this.nodes2.length; i++) matr.push(Array(this.nodes2.length).fill(0))
      this.links2.forEach(link => {
        matr[+link.source.id][+link.target.id] = +link.name;
        matr[+link.target.id][+link.source.id] = +link.name;
      })
      this.matrix2 = matr;
    },

    makeFO(matrix){
      let stringFO = '';
      matrix.forEach(el => {
        for(let i = 0; i < el.length; i++){
          if(el[i] !== 0) stringFO = stringFO + `${i},`;
        }
        stringFO = stringFO + '0,';
      })

      return stringFO;
    }
  },
}
</script>

<style src="vue-d3-network/dist/vue-d3-network.css">
</style>
<style>
#col-1 {
  position: relative;
  width: 50%;
  float: left;
  height: 100%;
  z-index: 1010101010
}

#col-2 {
  position: relative;
  width: 50%;
  float: left;
  height: 100%;
  z-index: 1010101010
}
</style>