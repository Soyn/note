* # Pivot Table In Dragonfly

  * #### **Pivot Table Data Structure**

```js
function PivotTable(row, col, pivot, wasDatasetRemoved) {
        this._sheet = null;
        this._pivot = pivot;
        this._wasDatasetRemoved = wasDatasetRemoved;

        this._oldCells = null;
        this._oldFormula = new Table();
        this._oldValue = new Table();
        this._oldStyles = new Table();
        this._row = row || 0;
        this._col = col || 0;
        this._dummyTable = {
            rowHeaderLevel: 1,
            colHeaderLevel: 1,
            rowCount: 10,
            colCount: 3,
        };
        this._emptyPivotTableImgUri = "/pivot/pivotTable/pivot_table_design.png";

        //new setting.
        this._theme = null;

        this._layoutSetting = {
            layout: pivotEnums.tableLayout.Tabular,
            subtotals: pivotEnums.subtotals.Bottom,
            grandTotals: pivotEnums.grandTotals.All,
            headerText: pivotEnums.headerText.NotRepeat,
            highlightOverwriteData: true
        };

        this._data = null;
        this._rowHeader = null;
        this._colHeader = null;
        this._grandTotoal = null;
        this._valueIdentifications = null;
    }
```

* #### Calculate Data In Pivot Table

```js
/*
* 这个函数主要用来做pivot table的数据映射
* 从pivot的instance中拿到数据然后
* 计算的结果用来初始化piovt table的数据成员
*/
calculate: function () {

                var self = this, ls = this._layoutSetting;

                if (!self._pivot || !self._pivot.hasData()) {
                    self._colHeaderLevel = 1;
                    self._rowHeaderLevel = 1;
                    self._dataRowCount = this._dummyTable.rowCount;
                    self._dataColCount = this._dummyTable.colCount;
                    return;
                }

                var result = self._pivot.getData(enums.scenarioType.Spread, this.getOption()),
                    tableData = result.data;

                this._rowHeaderLevel = result.schema.tableRowLevel + result.schema.rowLevel;
                this._colHeaderLevel = result.schema.tableColLevel + result.schema.colLevel + result.schema.valueLevel;
                this._valueHeaderLevel = result.schema.valueLevel;

                var drc = tableData.data.length;
                var dcc = tableData.data.reduce(function (ret, curr) { return Math.max(ret, curr.length); }, 0);

                var rhrc = tableData.rowHeader.length;
                var chcc = tableData.colHeader.reduce(function (ret, curr) { return Math.max(ret, curr.length); }, 0);

                this._dataRowCount = Math.max(drc, rhrc);
                this._dataColCount = Math.max(dcc, chcc);

                this._hasValueHeader = result.schema.valueLevel > 0;

                this._data = tableData.data;
                this._rowHeader = tableData.rowHeader;
                this._colHeader = tableData.colHeader;

                this._valueIdentifications = result.valueIdentifications;
            },
```

如下图，使用pivot中方的getData方法拿到当前pivoTtable中的pivot的数据，如data、schema、valueIdentifiction信息。

上面的代码中，最核心的部分就是getData里面的代码，getData的返回值如下图

![](/assets/pivot_table_getData.png)

getData的函数接收两个参数

```js
getData: function(scenarioType, option, /*isDvChart*/)
```

* Data数据包含了什么，这些数据是怎么计算的

```js
    function VectorProcesser(pivot, customManager) {
        this._pivot = pivot;
        this._pivotSetting = pivot.getSetting();
        this._customManager = customManager;

        var aggregated = this._pivot._aggregateData;
        var table = this._pivot._privateView.getDataSet().table;

        this._columns = getColumnsCache(aggregated, table);
        this._aggregated = aggregated;
        this._overwirteData = this._customManager.getOverwriteVectors();
    }
```



vectorProcessor是datset中处理数据的一个重要过程，上面是它的数据结构，数据的很多计算都是基于vector这个数据结构计算的，vector的来源有两种，一种来自于pivot setting，一种来自于customData，在计算的时候需要将这两个vector做一次merge：



```js
    function mergeAllVectors(vectors, customData, rankManager) {
        var vectorsObj = $.extend({}, vectors, customData.getMergedOverwriteVectors());
        var rankFilterFunc = rankManager.getVectorFilterFunc();
        return Object.keys(vectorsObj).filter(rankFilterFunc);
    }
```

```js
            var render = function (sheet, dashboardTheme) {
                var rowOffset = this._row, colOffset = this._col,
                    rc = this.getRowCount(), cc = this.getColCount();

                this.setTheme(new PivotTableTheme(this, dashboardTheme));

                expandSheet(sheet, rowOffset, colOffset, rc, cc);  // 重新计算pivot table应该占用的行列

                 this._theme.getAllNamedStyle().forEach(function (namedStyle) {
                     sheet.addNamedStyle(namedStyle);
                });
                // 如果pivot中有数据的话，render这个pivot table
                if (this._pivot && this._pivot.hasData()) {
                    this._renderTable(sheet);
                    this._hasData = true;
                } else {
                    this._renderEmptyTable(sheet);
                    this._hasData = false;
                }

                this._sheet = sheet;

                function expandSheet(sheet, row, col, rowCount, colcount) {
                    var totalRowCount = row + rowCount;
                    var totalColCount = col + colcount;
                    if (sheet.getRowCount() < totalRowCount) {
                        sheet.setRowCount(totalRowCount);
                    }
                    if (sheet.getColumnCount() < totalColCount) {
                        sheet.setColumnCount(totalColCount);
                    }
                }
            }
```

![](/assets/import.png)

如上图是在BuildPivotTbleHeader的时候每一个column的信息：

* key映射了真实的数据
* path
* vectors



实际上每一个node（前面计算好的colHeaderTree中的每一个node）的数据结构长下面这样

![](/assets/colheaderNode.png)

    

    在这里向node（前面计算好的colHeaderTree）中push从aggregators中读出来的值：

![](/assets/aggregator.png)

```js
                //render new pivot table area.
                scenario.scenarioData.forEachScenarioInfo(sheetId, function (info, row, col, index) {
                    var oldPivotTable = info.pivotTable;
                    if (needUpdate(oldPivotTable, updatedPivotIds)) {
                        var datasetId = info.data.getDataSetId();
                        var pivot = info.data.getPivot(dataSets.getDataViewCache(datasetId));
                        df.Calc.pivotCalc.addPivot(pivot);
                        var datasetCache = df.dataPackageService.getDatasetCache(datasetId),
                            datasetExist = datasetCache && datasetCache.deleted;
                        var pivotTable = new PivotTable(row, col, pivot, !datasetExist);
                        pivotTable.setPivotTableStyles(oldPivotTable.getPivotTableStyles()); //merge formatter.
                        pivotTable.setOption(oldPivotTable.getOption(), true);
                        pivotTable.calculate();
                        pivotTable.preRender(sheet, oldPivotTable);
                        pivotTable.render(sheet, dashboardTheme);
                        scenario.scenarioData.updatePivotTable(sheetId, pivotTable);

                        //update formula bar.
                        var range = pivotTable.getRange().union(oldPivotTable.getRange());
                        range && sheet.fireRangeChanged(range.row, range.col, range.rowCount, range.colCount);
                    }
                })
```

```js
  如何计算pivot table的数据：
```



