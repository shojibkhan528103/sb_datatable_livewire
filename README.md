To create a Livewire component table with the following features: searching, ascending/descending sorting, per-page item selection, pagination, and export options (PDF, Excel, Print), you can follow the steps below. I will guide you through the process step by step.

Step 1: Install Livewire and Required Packages
Ensure you have Livewire and required packages for PDF/Excel export installed in your Laravel project.

Install Livewire:

bash
Copy code
composer require livewire/livewire
Install Laravel Excel: This package is used to export to Excel and CSV.

bash
Copy code
composer require maatwebsite/excel
Install DomPDF: This package is used for exporting to PDF.

bash
Copy code
composer require barryvdh/laravel-dompdf
Step 2: Create a Livewire Component
Run the following command to generate a new Livewire component for the table:

bash
Copy code
php artisan make:livewire TableComponent
This will generate two files:

app/Http/Livewire/TableComponent.php
resources/views/livewire/table-component.blade.php
Step 3: Add the Table Logic in the Livewire Component
In the TableComponent.php file, define the table, sorting, pagination, search, and export logic.

php
Copy code
<?php

namespace App\Http\Livewire;

use Livewire\Component;
use Livewire\WithPagination;
use App\Models\YourModel; // Replace with your model
use Maatwebsite\Excel\Facades\Excel;
use App\Exports\YourModelExport;
use Barryvdh\DomPDF\Facade\Pdf;

class TableComponent extends Component
{
    use WithPagination;

    public $search = '';
    public $sortBy = 'id';
    public $sortDirection = 'asc';
    public $perPage = 10;

    public function updatingSearch()
    {
        $this->resetPage();
    }

    public function sortBy($field)
    {
        if ($this->sortBy === $field) {
            $this->sortDirection = $this->sortDirection === 'asc' ? 'desc' : 'asc';
        } else {
            $this->sortDirection = 'asc';
        }

        $this->sortBy = $field;
    }

    public function exportExcel()
    {
        return Excel::download(new YourModelExport, 'data.xlsx');
    }

    public function exportPDF()
    {
        $data = YourModel::all();
        $pdf = Pdf::loadView('exports.pdf', compact('data'));
        return $pdf->download('data.pdf');
    }

    public function render()
    {
        $items = YourModel::where('name', 'like', '%' . $this->search . '%') // Adjust based on your columns
            ->orderBy($this->sortBy, $this->sortDirection)
            ->paginate($this->perPage);

        return view('livewire.table-component', [
            'items' => $items,
        ]);
    }
}
Step 4: Create the View for the Table
In the table-component.blade.php, add the HTML for your table, pagination, and search box. Also, add export buttons for PDF, Excel, and Print.

blade
Copy code
<div>
    <div class="mb-4">
        <input type="text" wire:model="search" placeholder="Search..." class="form-input">
    </div>

    <div class="mb-4">
        <select wire:model="perPage" class="form-select">
            <option value="5">5 per page</option>
            <option value="10">10 per page</option>
            <option value="25">25 per page</option>
            <option value="50">50 per page</option>
        </select>
    </div>

    <table class="table-auto w-full">
        <thead>
            <tr>
                <th wire:click="sortBy('id')" class="cursor-pointer">ID</th>
                <th wire:click="sortBy('name')" class="cursor-pointer">Name</th>
                <th wire:click="sortBy('created_at')" class="cursor-pointer">Created At</th>
            </tr>
        </thead>
        <tbody>
            @foreach($items as $item)
            <tr>
                <td>{{ $item->id }}</td>
                <td>{{ $item->name }}</td>
                <td>{{ $item->created_at }}</td>
            </tr>
            @endforeach
        </tbody>
    </table>

    <div class="mt-4">
        {{ $items->links() }}
    </div>

    <div class="mt-4">
        <button wire:click="exportExcel" class="btn btn-primary">Export Excel</button>
        <button wire:click="exportPDF" class="btn btn-secondary">Export PDF</button>
        <button onclick="window.print()" class="btn btn-success">Print</button>
    </div>
</div>
Step 5: Create the Export Class for Excel
Generate the export class for Excel.

bash
Copy code
php artisan make:export YourModelExport --model=YourModel
Inside YourModelExport.php, configure the export logic:

php
Copy code
<?php

namespace App\Exports;

use App\Models\YourModel;
use Maatwebsite\Excel\Concerns\FromCollection;

class YourModelExport implements FromCollection
{
    public function collection()
    {
        return YourModel::all();
    }
}
Step 6: Create the PDF View (Optional)
In resources/views/exports/pdf.blade.php, define how the data should be formatted in the PDF:

blade
Copy code
<!DOCTYPE html>
<html>
<head>
    <title>Data Export PDF</title>
</head>
<body>
    <h1>Data Export</h1>
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Created At</th>
            </tr>
        </thead>
        <tbody>
            @foreach($data as $item)
            <tr>
                <td>{{ $item->id }}</td>
                <td>{{ $item->name }}</td>
                <td>{{ $item->created_at }}</td>
            </tr>
            @endforeach
        </tbody>
    </table>
</body>
</html>
Step 7: Include the Livewire Component in a Blade Template
Finally, include the Livewire component in any of your Blade views where you want to display the table.

blade
Copy code
<livewire:table-component />
Step 8: Run the Project
Run your Laravel project and view the table with sorting, pagination, searching, and export options.

bash
Copy code
php artisan serve
Conclusion
You now have a working Livewire component table that includes searching, sorting, per-page selection, pagination, and export functionality (PDF, Excel, Print).
