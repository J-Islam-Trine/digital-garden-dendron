---
id: jh10gs3w6l4d3i7ryg1i6jn
title: Sleep Chart in Chartview Obsidian
desc: ''
tags: [RAW]
updated: 1685693616186
created: 1685693593427
---

```chartsview
#-----------------#
#- chart type    -#
#-----------------#
type: Line

#-----------------#
#- chart data    -#
#-----------------#
data: |
  dataviewjs:
  return dv.pages('"Daily Notes"')
           .map(page => ({date: page.file.cday.toFormat("dd-MM-yy"), sleep_score: page.file.frontmatter.sleep_score, sleep_start: page.file.frontmatter.sleep_start, sleep_end: page.file.frontmatter.sleep_end}))
           .array();

#-----------------#
#- chart options -#
#-----------------#
options:
  tooltip:
    fields: ['sleep_score', 'sleep_start', 'sleep_end']
  xField: "date"
  yField: ['sleep_score']
  padding: auto
  label:
    position: "middle"
    style:
      opacity: 0.6
      fontSize: 12
  columnStyle:
    fillOpacity: 0.5
    lineWidth: 1
    strokeOpacity: 0.7
    shadowColor: "white"
    shadowBlur: 10
    shadowOffsetX: 5
    shadowOffsetY: 5
  xAxis:
    label:
      autoHide: false
      autoRotate: true
```