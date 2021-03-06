<template>
  <div class="grid-container">
    <template v-for="(pipeline, index) in pipelines">
      <div :key="index" v-if="!pipeline.p.notvalid">
        <div class="notification pipeline-box content-article">
          <div class="status-display-success" v-if="pipeline.r.status === 'success'"></div>
          <div class="status-display-fail" v-else-if="pipeline.r.status === 'failed'"></div>
          <div class="status-display-unknown" v-else></div>
          <div class="outer-box">
            <router-link :to="{ path: '/pipeline/detail', query: { pipelineid: pipeline.p.id }}" class="hoveraction">
              <div class="outer-box-icon-image">
                <img :src="getImagePath(pipeline.p.type)" v-bind:class="pipelineImageClass(pipeline.p.type)">
              </div>
              <div>
                <span class="subtitle">{{ pipeline.p.name }}</span>
              </div>
            </router-link>
            <hr class="pipeline-hr">
            <div>
              <i class="fa fa-hourglass"></i>
              <span style="color: #b1adad;">
                Duration:
              </span>
              <span v-if="pipeline.r.status === 'success' || pipeline.r.status === 'failed'">
                {{ calculateDuration(pipeline.r.startdate, pipeline.r.finishdate) }}
              </span>
              <span v-else>
                unknown
              </span><br/>
              <i class="fa fa-calendar"></i>
              <span style="color: #b1adad;">
                Started:
              </span>
              <span v-if="pipeline.r.status === 'success' || pipeline.r.status === 'failed'">
                {{ humanizedDate(pipeline.r.finishdate) }}
              </span>
              <span v-else>
                unknown
              </span><br/>
              <i class="fa fa-tags"></i>
              <span style="color: #b1adad;">
                Tags:
              </span>
              <span v-if="pipeline.p.tags">
                {{ $prettifyTags(pipeline.p.tags) }}
              </span>
              <span v-else>
                unknown
              </span><br/>
              <div class="pipelinegrid-footer">
                <a class="button is-primary" @click="checkPipelineArgsAndStartPipeline(pipeline.p)"
                   style="width: 100%;">
                  <span class="icon">
                    <i class="fa fa-play-circle"></i>
                  </span>
                  <span>Start Pipeline</span>
                </a>
              </div>
            </div>
          </div>
        </div>
      </div>
    </template>
    <div v-if="pipelines.length == 0" class="no-pipelines-div">
      <span class="no-pipelines-text">No pipelines are available. Please create a pipeline first.</span>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import helper from '../../helper'

export default {
  data () {
    return {
      pipelines: [],
      pipeline: null
    }
  },

  mounted () {
    // Fetch data from backend
    this.fetchData()

    // periodically update dashboard
    let intervalID = setInterval(function () {
      this.fetchData()
    }.bind(this), 3000)

    // Append interval id to store
    this.$store.commit('appendInterval', intervalID)
  },

  destroyed () {
    this.$store.commit('clearIntervals')
  },

  watch: {
    '$route': 'fetchData'
  },

  methods: {
    fetchData () {
      this.$http
        .get('/api/v1/pipeline/latest', {
          params: {
            hideProgressBar: true
          }
        })
        .then(response => {
          if (response.data) {
            this.pipelines = response.data
          }
        })
        .catch((error) => {
          this.$store.commit('clearIntervals')
          this.$onError(error)
        })
    },

    pipelineImageClass (type) {
      switch (type) {
        case 'python':
          return 'outer-box-image-python'
        case 'cpp':
          return 'outer-box-image-cpp'
        case 'ruby':
          return 'outer-box-image-ruby'
        case 'nodejs':
          return 'outer-box-image-nodejs'
        default:
          return 'outer-box-image'
      }
    },

    checkPipelineArgsAndStartPipeline (pipeline) {
      helper.StartPipelineWithArgsCheck(this, pipeline)
    },

    getImagePath (type) {
      return require('../../assets/images/' + type + '.png')
    },

    calculateDuration (startdate, finishdate) {
      if (moment(startdate).valueOf() < 0) {
        startdate = moment()
      }
      if (moment(finishdate).valueOf() < 0) {
        finishdate = moment()
      }

      // Calculate difference
      var diff = moment(finishdate).diff(moment(startdate), 'seconds')
      if (diff < 60) {
        return diff + ' seconds'
      }
      return moment.duration(diff, 'seconds').humanize()
    },

    humanizedDate (date) {
      return moment(date).format('LLL')
    }
  }
}
</script>

<style lang="scss">

  .grid-container {
    display: grid;
    grid-template-columns: repeat(auto-fill, 400px);
    grid-row-gap: 10px;
    grid-column-gap: 10px;
  }

  .no-pipelines-div {
    width: 100%;
    text-align: center;
    margin-top: 50px;
  }

  .no-pipelines-text {
    color: whitesmoke;
    font-size: 2rem;
  }

  @mixin status-display {
    position: fixed;
    min-width: 50px;
    height: 100%;
    margin-left: -23px;
    margin-top: -20px;
    margin-bottom: -20px;
    border-top-left-radius: 3px;
    border-bottom-left-radius: 3px;
    margin-right: 10px;
  }

  .status-display-success {
  @include status-display();
    background-color: rgb(49, 196, 49);
  }

  .status-display-folder {
  @include status-display();
    background-color: #4da2fc;
  }

  .status-display-fail {
  @include status-display();
    background-color: #ca280b;
  }

  .status-display-unknown {
  @include status-display();
    background-color: grey;
  }

  .pipeline-box {
    padding-right: 20px;
  }

  .outer-box {
    padding-left: 40px;
    min-height: 170px;
    width: 100%;
  }

  .outer-box-icon {
    width: 50px;
    float: left;
  }

  .outer-box-icon-image {
    float: left;
    width: 40px;
    height: 40px;
    overflow: hidden;
    border-radius: 50%;
    position: relative;
    border-color: whitesmoke;
    border-style: solid;
    margin-right: 10px;
    margin-top: -5px;
  }

  .outer-box-image {
    position: absolute;
    width: 50px;
    height: 50px;
    top: 70%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .outer-box-image-python {
    position: absolute;
    width: 50px;
    height: 40px;
    top: 53%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .outer-box-image-cpp {
    position: absolute;
    width: 27px;
    height: 30px;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .outer-box-image-nodejs {
    position: absolute;
    width: 30px;
    height: 30px;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .outer-box-image-ruby {
    position: absolute;
    width: 50px;
    height: 35px;
    top: 43%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .hoveraction:hover .outer-box-icon-image {
    border-color: #4da2fc !important;
  }

  .hoveraction:hover .subtitle {
    color: #4da2fc !important;
    text-decoration: underline;
    text-decoration-color: #4da2fc !important;
  }

  .pipeline-hr {
    background-position: bottom;
    background-size: 3px 1px;
    background-repeat: repeat-x;
  }

  .pipelinegrid-footer {
    padding-top: 20px;
  }

</style>
