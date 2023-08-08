# 结合html2canvas实现openlayers中区域地图导出_openlayers出图_牛老师讲GIS的博客-CSDN博客
GIS的“5M”应用中，有一个非常重要的应用领域——制图（Mapping）。然而现在的趋势是webgis的应用更为广泛，如何在web端实现地图的制图、导出与打印就是一个非常有用的功能，本文将结合html2canvas实现区域地图的导出。

![](https://img-blog.csdnimg.cn/img_convert/354c397e9d32462e905006ea9f577d7d.png)

![](https://img-blog.csdnimg.cn/img_convert/a071917cefe4532109d36f24f37407fc.png)

```js
const dragPanInteraction = map
  .getInteractions()
  .getArray()
  .find(interaction => {
    return interaction instanceof ol.interaction.DragPan
  })
const dblClickZoomInteraction = map
  .getInteractions()
  .getArray()
  .find(interaction => {
    return interaction instanceof ol.interaction.DoubleClickZoom
  })
const mouseWheelZoomInteraction = map
  .getInteractions()
  .getArray()
  .find(interaction => {
    return interaction instanceof ol.interaction.MouseWheelZoom
  })


function base64Img2Blob(code){
  var parts = code.split(';base64,');
  var contentType = parts[0].split(':')[1];
  var raw = window.atob(parts[1]);
  var rawLength = raw.length;
  var uInt8Array = new Uint8Array(rawLength);
  for (var i = 0; i < rawLength; ++i) {
    uInt8Array[i] = raw.charCodeAt(i);
  }
  return new Blob([uInt8Array], {type: contentType});
}

function downloadFile(fileName, content){
  var aLink = document.createElement('a');
  var blob = base64Img2Blob(content); 
  var evt = document.createEvent("HTMLEvents");
  evt.initEvent("click", false, false);
  aLink.download = fileName;
  aLink.href = URL.createObjectURL(blob);
  aLink.dispatchEvent(evt);
  aLink.click()
}

function setZoomVisible(display) {
  setDomDisplay('.ol-zoom', display)
}

function setDomDisplay(dom, display) {
  document.querySelector(dom).style.display = display
}

function setNorthArrowVisible(display) {
  setDomDisplay('.north-arrow', display)
}

function getSizeForZoom(extent, zoom) {
  const res = map.getView().getResolutionForZoom(zoom)
  
  const [paddingx, paddingy] = [100, 100]
  const [xmin, ymin, xmax, ymax] = extent
  const w = (xmax - xmin) / res + paddingx * 2;
  const h = (ymax - ymin) / res + paddingy * 2;
  return [w, h]
}

document.getElementById('setting').addEventListener('click', () => {
  
  const coords = [ [ [ 113.99364, 22.52712 ], [ 113.99817, 22.52795 ], [ 113.99867, 22.52494 ], [ 113.99819, 22.52449 ], [ 113.99463, 22.52253 ], [ 113.99439, 22.52256 ], [ 113.99364, 22.52712 ] ] ]
  const geom = new ol.geom.Polygon(coords)
  geom.transform('EPSG:4326', 'EPSG:3857')
  vectorSource.clear()
  const feature = new ol.Feature({
    geometry: geom
  })
  vectorSource.addFeature(feature)
  const extent = geom.getExtent()
  
  let w, h, z;
  const max = 3000
  for (let i = 18;i>10;i--) {
    const [_w, _h] = getSizeForZoom(extent, i)
    if(_w < max || _h < max) {
      z = i
      w = _w
      h = _h
      break;
    }
  }
  const view = map.getView()
  view.setZoom(z)
  const div = map.getTargetElement()
  div.style.width = w + 'px'
  div.style.height = h + 'px'
  const size = [w, h]
  map.setSize(size)
  view.fit(geom, {
    size: size
  })
  
  dragPanInteraction.setActive(false)
  dblClickZoomInteraction.setActive(false)
  mouseWheelZoomInteraction.setActive(false)

  
  const mapTitle = document.querySelector('.map-title')
  const title = mapTitle.innerText
  const canvas = map.getTargetElement().querySelector('canvas')
  const ctx = canvas.getContext('2d')
  ctx.font="18px 微软雅黑 bold";
  const tw = ctx.measureText(title)
  mapTitle.style.left = `calc(50% - ${tw.width / 2}px)`

  
  setDomDisplay('.ol-zoom', 'none')
  setDomDisplay('.north-arrow', 'block')
  setDomDisplay('.map-title', 'block')
  setDomDisplay('.map-info', 'block')
})

document.getElementById('export').addEventListener('click', () => {
  const div = map.getTargetElement()
  html2canvas(div).then(canvas => {
    downloadFile('map.png', canvas.toDataURL("image/png"));
  });
})

document.getElementById('print').addEventListener('click', () => {
  document.querySelector('.tools').style.display = 'none'
  window.print()
})

```