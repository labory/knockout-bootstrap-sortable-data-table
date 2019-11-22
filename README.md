Knockout pageable and sortable data table
=============================

### Basic
![Basic Example](https://raw.github.com/labory/knockout-bootstrap-sortable-data-table/master/assets/basic-example.png)
### View

    <table data-bind="dataTable: tableViewModel"><!-- --></table>

### ViewModel

    $(function () {
        var ExamplePageViewModel = function() {
            var self = this;
            self.getData = function (page, size, sortField, sortOrder, callback) {
                var params = "?page.page=" + (page) + "&page.size=" + size
                  + "&page.sort=" + sortField + "&page.sort.dir=" + sortOrder;
                $.getJSON("/frameworks" + params, callback);
            };
            self.tableViewModel = new ko.dataTable.ViewModel({
                columns: [
                    { name: "FRAMEWORK", value: "name", sortField: "name", width: "150px" },
                    { name: "LICENSE", value: "license", sortField: "license", width: "150px" },
                    { name: "DEPENDENCY", value: "dependency", sortField: "dependency", width: "150px" },
                    { name: "SIZE (Gzipped)", value: function (item) { return item.size != '' ? item.size + ' kb' : ''},
                      sortField: "size", width: "150px"}
                ],
                sortable: true,
                loader: self.getData,
                pageSize: 10
            });
        };
        ko.applyBindings(new ExamplePageViewModel());
    });

### Row Action Example
![Row Action Example](https://raw.github.com/labory/knockout-bootstrap-sortable-data-table/master/assets/data-row-action-example.png)
### View

    <table data-bind="dataTable: tableViewModel"><!-- --></table>

    <script type="text/html" id="actionColumnTemplate">
        <![CDATA[
            <button class="btn btn-info" type="submit" data-bind="click: function(data, event) {
                $root.like(data, event)}, clickBubble: false">Like</button>
        ]]>
    </script>

### ViewModel

    $(function () {
        var ExamplePageViewModel = function() {
            var self = this;
            self.selectedFramework = ko.observable();
            self.getData = function (page, size, sortField, sortOrder, callback) {
                var params = "?page.page=" + (page) + "&page.size=" + size
                  + "&page.sort=" + sortField + "&page.sort.dir=" + sortOrder;
                $.getJSON("/frameworks" + params, callback);
            };
            self.tableViewModel = new ko.dataTable.ViewModel({
                columns: [
                    { name: "FRAMEWORK", value: "name", sortField: "name", width: "25%"},
                    { name: "LICENSE", value: "license", sortField: "license", width: "25%"},
                    { name: "DEPENDENCY", value: "dependency", sortField: "dependency", width: "25%" },
                    { name: "SIZE (Gzipped)", value: function (item) { return item.size != '' ? item.size + ' kb' : ''}
                        , sortField: "size", width: "15%" },
                    { name: "ACTION", value: function (item) {return item}, template: "actionColumnTemplate", width: "10%"}
                ],
                sortable: true,
                loader: self.getData,
                selectItem: self.selectedFramework,
                pageSize: 10
            });
            self.tableViewModel.like =  function(framework, event) {
                console.log("your code goes here: " + framework.name + " , " + event.type);
                console.log("e.g. update data by ajax and reload table preserving selection");
                $.ajax({
                    url:'/frameworks/like',
                    type:'POST',
                    data:{'id': framework.id}
                }).success(function (messages) {
                    self.tableViewModel.reload(true /** true if selection should be preserved on data reload */);
                });
            };
        };
        ko.applyBindings(new ExamplePageViewModel());
    });
    
### Possible Backend Implementation (Spring Data)

    @RequestMapping(value = "/frameworks", method = RequestMethod.GET, produces = "application/json")
    public @ResponseBody Page<Framework> getFrameworks(@PageableDefaults(
            pageNumber = 0, value = 10, sort = "name", sortDir = Sort.Direction.ASC) Pageable pageable) {
        return frameworkRepository.findAll(pageable);
    }

    
### Dependencies
  * Knockout
  * Jquery
  * bootstrap pagination css
