---
title: Row Selection
page_title: Grid - Rows Selection
description: Learn how to select row in Blazor Grid component. Explore the selected rows. Discover row selection bevahior when combined with other Grid features. Try the practical sample code for row selection.
slug: grid-selection-row
tags: telerik,blazor,grid,selection,rows
previous_url: /components/grid/selection/single,/components/grid/selection/multiple
position: 3
---

# Row Selection

The Grid component offers support for single or multiple row selection. You can select a row with mouse click and through checkbox column. You can access the collection of selected rows, use this collection and manipulate it. You can follow and respond to the event of selection.

## Rows Selection Options

By default, users can select rows by clicking anywhere in the row, except on the command buttons.

To select multiple rows, hold down the `Ctrl` or `Shift` key to extend the selection:
* Press and hold `Ctrl` and click the desired rows to select or deselect them.
* Click on the starting row in a range of rows that you want to select, press and hold `Shift`, and click on the last row in the range. The first selected row is the starting point of the range and the last selected row is the end of the selection.

If you release the `Ctrl` or the `Shift` keys and click to start new multiple selection, the previously selected rows will be deselected.

You can also render a checkbox column that allows users to select and deselect rows. To use checkbox selection, add a [`GridCheckboxColumn`]({%slug components/grid/columns/checkbox%}) in the `GridColumns` collection of the Grid. The `GridCheckboxColumn` provides [additional configuration settings related to selection]({%slug components/grid/columns/checkbox%}#parameters).

You can combine click and checkbox selection or use only one selection option.

You can use row selection with both [selection modes]({%slug grid-selection-overview%}#use-single-or-multiple-selection)—single and multiple.

>caption Click selection and single selection mode

````CSHTML
Click on one row to select it.

<TelerikGrid Data=@GridData
             SelectionMode="@GridSelectionMode.Single"
             @bind-SelectedItems="@SelectedEmployees"
             Pageable="true">
    <GridColumns>
        <GridColumn Field="@nameof(Employee.Name)" />
        <GridColumn Field="@nameof(Employee.Team)" Title="Team" />
    </GridColumns>
</TelerikGrid>

<h2>Selected Employee:</h2>
<ul>
    @foreach (Employee employee in SelectedEmployees)
    {
        <li>@employee.Name</li>
    }
</ul>


@code {
    private List<Employee> GridData { get; set; }
    private IEnumerable<Employee> SelectedEmployees { get; set; } = Enumerable.Empty<Employee>();

    protected override void OnInitialized()
    {
        GridData = new List<Employee>();
        for (int i = 0; i < 15; i++)
        {
            GridData.Add(new Employee()
                {
                    EmployeeId = i,
                    Name = "Employee " + i.ToString(),
                    Team = "Team " + i % 3
                });
        }
    }

    public class Employee
    {
        public int EmployeeId { get; set; }
        public string Name { get; set; }
        public string Team { get; set; }
    }
}
````

## Selected Rows

* You can get or set the selected rows through the `SelectedItems` property. It is a collection of the [Grid's `Data`]({%slug grid-data-binding%}) model.
* You can use the `SelectedItems` collection in two-way binding. You can predefine the selected item for your users through the two-way binding of the `SelectedItems` property. The collection will be updated by the Grid when the selection changes.

### Selected Rows When Data Changes

When the Grid `Data` collection changes, the `SelectedItems` collection has the following behavior:

* When you update a selected item in the Grid, you have to make the same update in the `SelectedItems` collection through the [Grid `OnUpdate` editing event]({%slug components/grid/editing/overview%}#events).
* When you delete a selected item in the Grid, it will automatically delete from the `SelectedItems` collection. If you are using one-way binding for the `SelectedItems` collection and the [`SelectedItemsChanged` event](#selecteditemschanged), when you delete a selected item, the event fires. When you delete all selected items, the `SelectedItemsChanged` event fires with an empty collection.
* When you create an item in the Grid, and you want to select it at the same time, use the [`OnCreate` event]({%slug components/grid/editing/overview%}#events).

### Selected Rows Equals Comparison

The `SelectedItems` collection is compared against the Grid `Data` collection in order to determine which rows will be highlighted. The default behavior of the framework is to compare objects by their reference.

When the `SelectedItems` are obtained from a different data source to the Grid (e.g., from a separate service method and not from the view-model), the references may not match and so there will be no highlighted items. In such cases, you have to [override the `Equals` method of the underlying model class]({%slug grid-state%}#equals-comparison) so that it matches them, for example, by a unique identifier rather than by reference so that two objects can be equal regardless of their origin, but according to their contents. When you are overriding the `Equals` method, it is also recommended to override the [`GetHashCode`](https://docs.microsoft.com/en-us/dotnet/api/system.object.gethashcode) method as well.

## SelectedItemsChanged

You can respond to the user action of selecting a new row through the `SelectedItemsChanged` event. The `SelectedItemsChanged` event receives a collection of the Grid data model. It may have no items in it. It may have only one member (the last selected item) when the `SelectionMode` is `Single`.

>caption Single selection with one-way binding for SelectedItems and using the SelectedItemsChanged event

````CSHTML
@* Select a single row and follow the selection event*@

<TelerikGrid Data=@GridData
             SelectionMode="@GridSelectionMode.Single"
             SelectedItems="@SelectedEmployees"
             SelectedItemsChanged="@((IEnumerable<Employee> employeeList) => OnRowSelect(employeeList))"
             Pageable="true"
             Height="400px">
    <GridColumns>
        <GridCheckboxColumn />
        <GridColumn Field="@nameof(Employee.Name)" />
        <GridColumn Field="@nameof(Employee.Team)" Title="Team" />
    </GridColumns>
</TelerikGrid>

<h2>Selected Employee:</h2>
<ul>
    @foreach (Employee employee in SelectedEmployees)
    {
        <li>@employee.Name</li>
    }
</ul>


@code {
    private List<Employee> GridData { get; set; } = new();
    private IEnumerable<Employee> SelectedEmployees { get; set; } = Enumerable.Empty<Employee>();
    private Employee? SelectedEmployee { get; set; }

    protected override void OnInitialized()
    {
        GridData = new List<Employee>();
        for (int i = 1; i < 15; i++)
        {
            GridData.Add(new Employee()
                {
                    EmployeeId = i,
                    Name = $"Employee {i}",
                    Team = "Team " + i % 3
                });
        }
    }

    protected void OnRowSelect(IEnumerable<Employee> employees)
    {
        SelectedEmployee = employees.FirstOrDefault();
        // update the collection so that the grid can highlight the correct item
        // when two-way binding is used this happens automatically,
        // but the framework does not allow two-way binding and the event at the same time
        SelectedEmployees = new List<Employee> { SelectedEmployee };
    }

    public class Employee
    {
        public int EmployeeId { get; set; }
        public string Name { get; set; }
        public string Team { get; set; }
    }
}

````

### SelectedItemsChanged and Asynchronous Operations

To execute asynchronous operations, such as loading data on demand, when the user selects a row, handle these operations in the [`OnRowClick`]({%slug grid-events%}#onrowclick) or [`OnRowDoubleClick`]({%slug grid-events%}#onrowdoubleclick) events rather than in the [`SelectedItemsChanged`]({%slug grid-events%}#selecteditemschanged).

## Row Selection and Other Grid Features

The selection feature behavior may vary when the Grid configuration combines row selection and other Grid features, such as editing, virtualization, paging, templates, row drag and drop. In such cases you need to consider certain limitation or include some modications.

### Selection and Editing Modes

When you want to edit a row, the row selection has the following behavior:

* In the [Incell EditMode]({%slug components/grid/editing/incell%}) the row selection can be applied only through a [checkbox column](#checkbox-selection) (`<GridCheckboxColumn />`). This applies for both selection modes—single and multiple. This is required due to the overlapping action that triggers selection and InCell editing (clicking in the row). Otherwise, if the row click selection is enabled with InCell editing, each attempt to select a row would put a cell in edit mode; and each attempt to edit a cell would select a new row. Such user experience is confusing, and so the row selection can be applied only through checkbox column when there is InCell editing mode. To see how to select the row that is being edited in InCell edit mode without using a `<GridCheckboxColumn />` check out the [Row Selection in Edit with InCell EditMode]({%slug grid-kb-row-select-incell-edit%}) Knowledge Base article.
* In [Inline EditMode]({%slug components/grid/editing/inline%}) and [Popup EditMode]({%slug components/grid/editing/popup%}) the row selection can be done by clicking on the desired row or by using a `<GridCheckboxColumn />`.

### Selection and Virtual Scrolling

When the Grid has [virtual scrolling]({%slug components/grid/virtual-scrolling%}) and the `SelectionMode` is set to [`Multiple`](#selection-mode) the selectable rows will be the one in the current set of items (page). If you select a row and scroll down to some of the ones that are not rendered yet (virtualization kicks in) and you want to select that range with the `Shift` button, the selection will start from the position of the first item of the current set (page) to the last selected row.

### Selection and Paging

The `SelectedItems` collection persists across paging operations. Changing the page will keep it populated and you can add more items to the collection.

### Selection and Templates

When your Grid configuration contains [Grid templates]({%slug components/grid/features/templates%}) and row selection:

* If you are using a [Grid Column Template]({%slug grid-templates-column%}) and you have a clickable component as content of the Grid Column Template, you can check the knowledge base article on [how to stop the row selection from being triggered when the user clicks another component in the Grid Column Template]({%slug grid-kb-row-selection-in-column-template%}).
* If you are using the [Row Template]({%slug components/grid/features/templates%}#row-template) and you want to select a row via a [checkbox column](#checkbox-selection) (`<GridCheckboxColumn />`) the Grid cannot render selection checkboxes for you. You have to bind them yourself to a field in the model, and handle their selection changed event to populate the `SelectedItems` collection of the Grid. You can find an example to get started in the following thread: [Grid Row Template with Selection - Unsure how to Bind to Selected Item](https://feedback.telerik.com/blazor/1463819-grid-row-template-with-selection-unsure-how-to-bind-to-selected-item).

### Selection and Row Drag and Drop

If the user drags selected rows, the current selection will be cleared on row drop.

## See Also

  * [Live Demo: Grid Row Selection](https://demos.telerik.com/blazor-ui/grid/row-selection)