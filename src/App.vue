<template>
  <div>
    <video width="400" height="300" id="video"></video>
    <canvas id="canvas">
    </canvas>

    <div>head1: {{head1}}</div>
    <div>hand1: {{hand1}}</div>
    <div>hand2: {{hand2}}</div>

  </div>
</template>
<script>
  import { filter, map } from "rxjs/operators";
  import { FaceEstimatorEvent } from "./vid/screen-gesture-control-master/src/libs/EstimateFace/eventEmitter";
  import { HandEstimator } from "./vid/screen-gesture-control-master/src/libs/EstimatteHand";
  import { HandEstimatorEvent } from "./vid/screen-gesture-control-master/src/libs/EstimatteHand/eventEmitter";
  import { HandCursor } from "./vid/screen-gesture-control-master/src/libs/EstimatteHand/features/handCursor";
  import { interpolation2dPoints } from "./vid/screen-gesture-control-master/src/libs/math/extrapolation";
  import { Position2D } from "./vid/screen-gesture-control-master/src/libs/types/math";
  import {FaceEstimator} from "./vid/screen-gesture-control-master/src/libs/EstimateFace";


  export default {
    data(){
      return {
        head1:'',
        hand1:'',
        hand2:'',
      };
    },
    mounted() {

      document.addEventListener("DOMContentLoaded", async () => {
        const video = document.querySelector("video");
        if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
          return;
        }

       // video.srcObject = await navigator.mediaDevices.getUserMedia({ video: true });
        video.srcObject =   await navigator.mediaDevices.getUserMedia({ video: true })
         .catch(function(err) {
          console.log(err)
         });

        console.log('qwe')
        await video.play();

      //  this.runFaceWebInterface(video);
         this.runHandWebInterface(video);
      });

    },
    methods: {

      async runFaceWebInterface(video) {
        const estimator = new FaceEstimator(video, { updateTime: 1000 });
        await estimator.init();
        estimator
          .getEventEmitter()
          .createObserverOn(FaceEstimatorEvent.UPDATE)
          .pipe(
            filter((payload) => payload.headPitch < -5 || payload.headPitch > 5)
          )
          .subscribe((payload) => {
            this.head1=payload.headPitch
            console.log(payload.headPitch)
            // document.body.scrollTo({
            //     behavior: "smooth",
            //     top: document.body.scrollTop + payload.headPitch * 10,
            // });


          });
      },

      async  runHandWebInterface(video) {
        const estimator = new HandEstimator(video, { updateTime: 300 });
        await estimator.init();

        const cursor = new HandCursor();
        const interpolation = interpolation2dPoints(5);

        estimator
          .getEventEmitter()
          .createObserverOn(HandEstimatorEvent.UPDATE)
          .pipe(
            map((handEvent) =>
              interpolation(
                handEvent.indexFingerPoint[0],
                handEvent.indexFingerPoint[1],
                this.hand1=handEvent.indexFingerPoint[0],
                this.hand2=handEvent.indexFingerPoint[1]
              )
            ),
            filter(Boolean),
            map((point) => ({

              x: window.outerWidth - (window.outerWidth / video.videoWidth) * point.x,
              y: (window.outerHeight / video.videoHeight) * point.y,
            }))
          )
          .subscribe((point) => cursor.move(point));

        const gestureContainer = document.querySelector("#gesture");
        estimator
          .getEventEmitter()
          .createObserverOn(HandEstimatorEvent.GESTURE_UPDATE)
          .subscribe((gesture) => {
            gestureContainer.innerHTML = gesture.gestureType;
          });
      }
    }
  }

</script>
