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









