startTime(){
      let endTime = this.result.date.split(" ")[1]
      endTime = endTime.split(':').reduce((acc,time) => (60 * acc) + +time)
      let startTime = endTime - this.result.elapsed_time
      startTime = (new Date(startTime * 1000)).toUTCString().match(/(\d\d:\d\d:\d\d)/)[0]
      return startTime
    }
