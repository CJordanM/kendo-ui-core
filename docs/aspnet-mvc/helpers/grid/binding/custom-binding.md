---
title: Custom Binding
page_title: Custom Binding | Kendo UI Grid HtmlHelper
description: "Use and configure Kendo UI Grid for ASP.NET MVC for server custom binding."
previous_url: /aspnet-mvc/helpers/grid/custom-binding
slug: custombinding_grid_aspnetmvc
position: 3
---

# Custom Binding

Kendo Grid for ASP.NET MVC provides ability for the developer to bypass the built-in data processing and to handle operations such paging, sorting, filtering and grouping by himself. This is supported both for server and ajax binding.

## Custom Server Binding

To configure the Kendo Grid for server custom binding follow these steps:

1.  Add a new parameter of type `Kendo.UI.DataSourceRequest` to the action method.
It will contain the current grid request information - page, sort, group and filter.
Decorate that parameter with the `Kendo.UI.DataSourceRequestAttribute`. That attribute is responsible for populating the `DataSourceRequest` object.

        public ActionResult Index([DataSourceRequest(Prefix = "Grid")] DataSourceRequest request)
        {
            IQueryable<Order> orders = new NorthwindEntities().Orders;
        }
2.  Assign a default pageSize:

        public ActionResult Index([DataSourceRequest(Prefix = "Grid")] DataSourceRequest request)
        {
            if (request.PageSize == 0)
            {
               request.PageSize = 10;
            }

            IQueryable<Order> orders = new NorthwindEntities().Orders;
        }
3.  Handle the appropriate data operations:

        public ActionResult Index([DataSourceRequest(Prefix = "Grid")] DataSourceRequest request)
        {
            if (request.PageSize == 0)
            {
                request.PageSize = 10;
            }
            IQueryable<Order> orders = new NorthwindEntities().Orders;

            if (request.Sorts.Any())
            {
                foreach (SortDescriptor sortDescriptor in request.Sorts)
                {
                    if (sortDescriptor.SortDirection == ListSortDirection.Ascending)
                    {
                        switch (sortDescriptor.Member)
                        {
                            case "OrderID":
                                orders= orders.OrderBy(order => order.OrderID);
                                break;
                            case "ShipAddress":
                                orders= orders.OrderBy(order => order.ShipAddress);
                                break;
                        }
                    }
                    else
                    {
                        switch (sortDescriptor.Member)
                        {
                            case "OrderID":
                                orders= orders.OrderByDescending(order => order.OrderID);
                                break;
                            case "ShipAddress":
                                orders= orders.OrderByDescending(order => order.ShipAddress);
                                break;
                        }
                    }
                }
            }
            else
            {
                // EF can't page unsorted data
                orders = orders.OrderBy(o => o.OrderID);
            }

            // Apply paging
            if (request.Page > 0) {
                orders = orders.Skip((request.Page - 1) * request.PageSize);
            }
            orders = orders.Take(request.PageSize);
        }
4.  Calculate the total number of records.

        public ActionResult Index([DataSourceRequest(Prefix = "Grid")] DataSourceRequest request)
        {
            if (request.PageSize == 0)
            {
                request.PageSize = 10;
            }
            IQueryable<Order> orders = new NorthwindEntities().Orders;

            //Apply sorting (code omitted)

            // Calculate the total number of records before paging
            var total = orders.Count();

            // Apply paging

            ViewData["total"] = total;
        }
5.  Pass the processed data to the View.

        public ActionResult Index([DataSourceRequest]DataSourceRequest request)
        {
            // Get the data (code omitted)
            // Apply sorting (code omitted)
            // Calculate the total number of records (code omitted)

            // Apply paging (code omitted)

            return View(orders);
        }
6.  Set `EnableCustomBinding(true)` through the Grid Widget declaration.

        @model IEnumerable<KendoGridCustomServerBinding.Models.Order>

        @(Html.Kendo().Grid(Model)
            .Name("Grid")
            .EnableCustomBinding(true)
            .Columns(columns => {
                columns.Bound(o => o.OrderID);
                columns.Bound(o => o.ShipAddress);
            })
            .Pageable()
            .Sortable()
            .Scrollable()
        )
7.  Assign the total number of records through the `DataSource`, in case paging is enabled.

        @model IEnumerable<KendoGridCustomServerBinding.Models.Order>

        @(Html.Kendo().Grid(Model)
            .Name("Grid")
            .EnableCustomBinding(true)
            .Columns(columns => {
                columns.Bound(o => o.OrderID);
                columns.Bound(o => o.ShipAddress);
            })
            .Pageable()
            .Sortable()
            .Scrollable()
            .DataSource(dataSource => dataSource
                .Server()
                .Total((int)ViewData["total"]) // set the total number of records
            )
        )

[Download Visual Studio Project](https://github.com/telerik/ui-for-aspnet-mvc-examples/tree/master/grid/custom-server-binding)

## Custom Ajax Binding

To configure the Kendo Grid for ajax custom binding follow these steps:

1. Add a new parameter of type `Kendo.UI.DataSourceRequest` to the action method.
It will contain the current grid request information - page, sort, group and filter. Decorate that parameter with the
`Kendo.UI.DataSourceRequestAttribute`. That attribute is responsible for populating the DataSourceRequest object.

        public ActionResult Orders_Read([DataSourceRequest]DataSourceRequest request)
        {
            IQueryable<Order> orders = new NorthwindEntities().Orders;
        }
2. Handle the appropriate data operations and calculate the total number of records.

        public ActionResult Orders_Read([DataSourceRequest]DataSourceRequest request)
        {
            IQueryable<Order> orders = new NorthwindEntities().Orders;

            //Apply sorting

            if (request.Sorts.Any())
            {
                foreach (SortDescriptor sortDescriptor in request.Sorts)
                {
                    if (sortDescriptor.SortDirection == ListSortDirection.Ascending)
                    {
                        switch (sortDescriptor.Member)
                        {
                            case "OrderID":
                                orders= orders.OrderBy(order => order.OrderID);
                                break;
                            case "ShipAddress":
                                orders= orders.OrderBy(order => order.ShipAddress);
                                break;
                        }
                    }
                    else
                    {
                        switch (sortDescriptor.Member)
                        {
                            case "OrderID":
                                orders= orders.OrderByDescending(order => order.OrderID);
                                break;
                            case "ShipAddress":
                                orders= orders.OrderByDescending(order => order.ShipAddress);
                                break;
                        }
                    }
                }
            }
            else
            {
                // EF can't page unsorted data
                orders = orders.OrderBy(o => o.OrderID);
            }

            var total = orders.Count();

            // Apply paging
            if (request.Page > 0) {
                orders = orders.Skip((request.Page - 1) * request.PageSize);
            }
            orders = orders.Take(request.PageSize);
        }
3. Create new instance of `DataSourceResult` and set the `Data` and `Total` properties to the processed data and total number of records.

        public ActionResult Orders_Read([DataSourceRequest]DataSourceRequest request)
        {
            // Get the data (code omitted)
            IQueryable<Order> orders = new NorthwindEntities().Orders;

            // Apply sorting (code omitted)

            // Apply paging (code omitted)

            // Initialize  the DataSourceResult
            var result = new DataSourceResult()
            {
                Data = orders, // Process data (paging and sorting applied)
                Total = total // Total number of records
            };
        }
4. Return the `DataSourceResult` as JSON.

        public ActionResult Orders_Read([DataSourceRequest]DataSourceRequest request)
        {
            // Get the data (code omitted)
            IQueryable<Order> orders = new NorthwindEntities().Orders;

            // Apply sorting (code omitted)

            // Apply paging (code omitted)

            // Initialize  the DataSourceResult (code omitted)

            var result = new DataSourceResult()
            {
                Data = orders, // Process data (paging and sorting applied)
                Total = total // Total number of records
            };

            // Return the result as JSON
            return Json(result);
        }
5. Configure the grid for custom ajax binding

        @(Html.Kendo().Grid<KendoGridCustomAjaxBinding.Models.Order>()
            .Name("Grid")
            .EnableCustomBinding(true)
            .Columns(columns => {
                columns.Bound(o => o.OrderID);
                columns.Bound(o => o.ShipAddress);
            })
            .Pageable()
            .Sortable()
            .Scrollable()
            .DataSource(dataSource => dataSource
                .Ajax()
                .Read("Orders_Read", "Home")
            )
        )


## See Also

Other articles on the Kendo UI Grid for ASP.NET MVC:

* [Overview of the Grid HtmlHelper]({% slug overview_gridhelper_aspnetmvc %})
* [Configuration of the Grid HtmlHelper]({% slug configuration_gridhelper_aspnetmvc %})
* [Scaffolding]({% slug scaffoldinggrid_aspnetmvc %})
* [Excel Export]({% slug excelexport_gridhelper_aspnetmvc %})
* [Frequently Asked Questions]({% slug freqaskedquestions_gridhelper_aspnetmvc %})
* [Editing of the Grid HtmlHelper]({% slug ajaxediting_grid_aspnetmvc %})
* [Templating of the Grid HtmlHelper]({% slug clientdetailtemplate_grid_aspnetmvc %})
* [Troubleshooting of the Grid HtmlHelper]({% slug troubleshoot_gridhelper_aspnetmvc %})
* [API Reference of the Grid HtmlHelper](/api/aspnet-mvc/Kendo.Mvc.UI.Fluent/GridBuilder)
* [Overview of the Kendo UI Grid Widget]({% slug overview_kendoui_grid_widget %})

Articles on Telerik UI for ASP.NET MVC:

* [Overview of Telerik UI for ASP.NET MVC]({% slug overview_aspnetmvc %})
* [Fundamentals of Telerik UI for ASP.NET MVC]({% slug fundamentals_aspnetmvc %})
* [Scaffolding in Telerik UI for ASP.NET MVC]({% slug scaffolding_aspnetmvc %})
* [Telerik UI for ASP.NET MVC API Reference Folder](/api/aspnet-mvc/Kendo.Mvc/AggregateFunction)
* [Telerik UI for ASP.NET MVC HtmlHelpers Folder]({% slug overview_barcodehelper_aspnetmvc %})
* [Tutorials on Telerik UI for ASP.NET MVC]({% slug overview_timeefficiencyapp_aspnetmvc6 %})
* [Telerik UI for ASP.NET MVC Troubleshooting]({% slug troubleshooting_aspnetmvc %})
* [Download Visual Studio Project](https://github.com/telerik/ui-for-aspnet-mvc-examples/tree/master/grid/custom-ajax-binding)