# openlayers4+中高德导航路径的实现_getcoordinateat_牛老师讲GIS的博客-CSDN博客
许久未更新了，今天特此更新一篇，以示我还在，我也会一直在。今天的更新比较简单，就是在openlayers4+中实现类似于高德导航路径的样式。

![](https://img-blog.csdnimg.cn/img_convert/cc69b631f0b8f20c19bc1aec6fe9d77b.png)

1.  用`ol.layer.Vector`的`styleFunction`，返回一个`styleCollection`；
2.  `ol.geom.LineString`的`getCoordinateAt`接口实现线上等距的箭头展示；
3.  箭头方向通过`rotation`参数来控制，其计算公式是`const angle = (Math.atan2(point0[0] - point[0], point0[1] - point[1])) - Math.PI / 2`

```js
function styleFunc(feat) {
  let styles = []
  styles.push(new ol.style.Style({
    stroke: new ol.style.Stroke({
      color: '#089519',
      width: 6
    }),
    image: new ol.style.Icon({
      src:
        'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAADFklEQVRYR8WXS2gTURSG/zPt3BQF8bHx0UwixmZSH1AVVNqFVDeKC134Qhd1JbVuRBFBwQqKIIobFV2pYPHRhS5ENyouxAf4ALFNakWTadWV2lVp7tg5Mm0jSTqZV4vN8t7z/+ebc8+5lxCm+EdTnB8TAvhcG0nYH5Doz38O+yGBALKxafOG+M9BMJaCsBrA7LHEv8B4DcLHGqq+EM8N/vAL5BvgkxbZbIGvApjvYf5dAe2rM/IP/ED4Akhrop2AE34MCzEEtCcNedJL4wnQG1MbhpneeRk57VcRr1icM9+7aV0BvsYxU1riOQNLHEw6mPmxvU5EGwDsLo8hoEsosmlhFgOVIFwB0lH1HBEdchBv1w3ZWbye0cQ2AHfLY5n5fKrPPBwKIKOpTwBqLhYzeFXKMN86GaY1dSWB3pTu8VPdMNeHBBC/AcwsEnfqhtzudqYZTdhVsKtR+A3ohpwVGKAnLnS2kC4WEnNbss+87AbQE1X3M9GlEp2CVDIrM066ij3QFZ8+t8oySy4UZt6b6jOvuwGko2oLEV0rjlGrldiiL0NGIAA7OKOJFwDWFgk7dEPu8TiCm2UT0asbsi7wEdiCtCaOEnCmTDxuAgr7zpNAV3Qj3xoKoDuurlEsejlutBwmwXkCAAa3pAzzRiiAsWOwG1F3MOgk5mf2OhOtK+v8f+EmoX5ZTpY0c0mDup3nCEBUHAPhlFdchf07uiF3umm934IEZgxL8QpAKiBEvhrcmKhwaRW8PAFGq6C2gsh1/sfBEU7rOXncC9oXwGgvRJ4D3OhlaO/bj9CgIpsaXB6hQBWwg7ujYodCuO0HwM+FFRhgpAoxcQ+MLa4QhPt6Tm71AzpWLb+hQKZWLIeCpwDmVFD9hIVmvV9+8OvquwcKht2xSJvCfNEpgUV0oD6XL3mIvEACA9iGPTFxixkl802E28mc3OWVsHw/FMDXOGrylrD/CywYM/wWUWRiYRZD/wVgtAqRjcz8cKSRiDYlc/lHQZMHbsLyBBlNnLXXdEMeCZN8wgBhkxbrQvXAZCQOdRFNZuKC119mavcheswzygAAAABJRU5ErkJggg=='
    })
  }))
  const geometry = feat.get('geometry')
  if(geometry.getType().toLowerCase() === 'linestring') {
    const coords = geometry.getCoordinates()
    const length = geometry.getLength()
    const res = map.getView().getResolution() * 80
    const count = Math.ceil(length / res)
    
    for(let i = 1;i < count; i++) {
      const frag = i / count
      const point = geometry.getCoordinateAt(frag)
      const point0 = geometry.getCoordinateAt(frag + frag * 0.05)
      const angle = (Math.atan2(point0[0] - point[0], point0[1] - point[1])) - Math.PI / 2
      styles.push(new ol.style.Style({
        geometry: new ol.geom.Point(point),
        image: new ol.style.Icon({
          scale: 0.5,
          rotation: angle,
          snapToPixel: true,
          src:
            'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAAnElEQVQ4T63TsQ0CMQyF4f/NgMQQ0CBR0FIx190cFIiWhhFoKdgEiRUeSoF0gO8cjkub+Evs2OLPpc9422dgBRwlNZn/BtjeApdOUJsh0QuuwKYWiYAFcAKWHaSR1EbpfAHlkO0ICdMJgV+QXmAAmUu6v9IZA8wkPVKgtg7TF7H25t4UbN+A9ahGmqqVD8AO2GdzUF45+I3ZJJb9JxbwRhEhB66xAAAAAElFTkSuQmCC'
        })
      }))
    }
    
    styles.push(new ol.style.Style({
      geometry: new ol.geom.Point(coords[0]),
      image: new ol.style.Circle({
        radius: 12,
        fill: new ol.style.Fill({
          color: '#1677d8'
        })
      }),
      text: new ol.style.Text({
        offsetX: 0,
        offsetY: 0,
        textAlign: 'center',
        textBaseline: 'middle',
        text: '起',
        font: '12px sans-serif',
        fill: new ol.style.Fill({
          color: 'white'
        })
      })
    }))
    styles.push(new ol.style.Style({
      geometry: new ol.geom.Point(coords[coords.length - 1]),
      image: new ol.style.Circle({
        radius: 12,
        fill: new ol.style.Fill({
          color: '#bb1422'
        })
      }),
      text: new ol.style.Text({
        offsetX: 0,
        offsetY: 0,
        textAlign: 'center',
        textBaseline: 'middle',
        text: '终',
        font: '12px sans-serif',
        fill: new ol.style.Fill({
          color: 'white'
        })
      })
    }))
  }
  return styles
}

```