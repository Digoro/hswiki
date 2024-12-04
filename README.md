const displayStartTime = JulianDate.fromDate(new Date('1970-01-01'));
    const displayEndTime = JulianDate.fromDate(new Date('2300-01-01'));
    this.position = new SampledPositionProperty();
    this.satellite = this.viewer.entities.add({
      availability: new TimeIntervalCollection([new TimeInterval({
        start: displayStartTime,
        stop: displayEndTime
      })]),
      position: this.position,
      model: {
        uri: '/assets/satellite.glb',
        minimumPixelSize: 64,
        maximumScale: 50000,
        scale: 50000
      },
      path: {
        resolution: 1,
        material: new PolylineGlowMaterialProperty({
          glowPower: 0.2,
          color: Color.YELLOW
        }),
        width: 5
      }
    });
