# Documentation for widgets. File: 01-installation.md
---
title: Installation
---

**The Widgets package is pre-installed with the [Panel Builder](/docs/panels).** This guide is for using the Widgets package in a custom TALL Stack application (Tailwind, Alpine, Livewire, Laravel).

## Requirements

Filament requires the following to run:

- PHP 8.1+
- Laravel v10.0+
- Livewire v3.0+
- Tailwind v3.0+ [(Using Tailwind v4?)](#installing-tailwind-css)

## Installation

Require the Widgets package using Composer:

```bash
composer require filament/widgets:"^3.3" -W
```

## New Laravel projects

To quickly get started with Filament in a new Laravel project, run the following commands to install [Livewire](https://livewire.laravel.com), [Alpine.js](https://alpinejs.dev), and [Tailwind CSS](https://tailwindcss.com):

> Since these commands will overwrite existing files in your application, only run this in a new Laravel project!

```bash
php artisan filament:install --scaffold --widgets

npm install

npm run dev
```

## Existing Laravel projects

Run the following command to install the Widgets package assets:

```bash
php artisan filament:install --widgets
```

### Installing Tailwind CSS

> Filament uses Tailwind CSS v3 for styling. If your project uses Tailwind CSS v4, you will unfortunately need to downgrade it to v3 to use Filament. Filament v3 can't support Tailwind CSS v4 since it introduces breaking changes. Filament v4 will support Tailwind CSS v4.

Run the following command to install Tailwind CSS with the Tailwind Forms and Typography plugins:

```bash
npm install tailwindcss@3 @tailwindcss/forms @tailwindcss/typography postcss postcss-nesting autoprefixer --save-dev
```

Create a new `tailwind.config.js` file and add the Filament `preset` *(includes the Filament color scheme and the required Tailwind plugins)*:

```js
import preset from './vendor/filament/support/tailwind.config.preset'

export default {
    presets: [preset],
    content: [
        './app/Filament/**/*.php',
        './resources/views/filament/**/*.blade.php',
        './vendor/filament/**/*.blade.php',
    ],
}
```

### Configuring styles

Add Tailwind's CSS layers to your `resources/css/app.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
@tailwind variants;
```

Create a `postcss.config.js` file in the root of your project and register Tailwind CSS, PostCSS Nesting and Autoprefixer as plugins:

```js
export default {
    plugins: {
        'tailwindcss/nesting': 'postcss-nesting',
        tailwindcss: {},
        autoprefixer: {},
    },
}
```

### Automatically refreshing the browser
You may also want to update your `vite.config.js` file to refresh the page automatically when Livewire components are updated:

```js
import { defineConfig } from 'vite'
import laravel, { refreshPaths } from 'laravel-vite-plugin'

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: [
                ...refreshPaths,
                'app/Livewire/**',
            ],
        }),
    ],
})
```

### Compiling assets

Compile your new CSS and Javascript assets using `npm run dev`.

### Configuring your layout

Create a new `resources/views/components/layouts/app.blade.php` layout file for Livewire components:

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">

        <meta name="application-name" content="{{ config('app.name') }}">
        <meta name="csrf-token" content="{{ csrf_token() }}">
        <meta name="viewport" content="width=device-width, initial-scale=1">

        <title>{{ config('app.name') }}</title>

        <style>
            [x-cloak] {
                display: none !important;
            }
        </style>

        @filamentStyles
        @vite('resources/css/app.css')
    </head>

    <body class="antialiased">
        {{ $slot }}

        @filamentScripts
        @vite('resources/js/app.js')
    </body>
</html>
```

## Publishing configuration

You can publish the package configuration using the following command (optional):

```bash
php artisan vendor:publish --tag=filament-config
```

## Upgrading

Filament automatically upgrades to the latest non-breaking version when you run `composer update`. After any updates, all Laravel caches need to be cleared, and frontend assets need to be republished. You can do this all at once using the `filament:upgrade` command, which should have been added to your `composer.json` file when you ran `filament:install` the first time:

```json
"post-autoload-dump": [
    // ...
    "@php artisan filament:upgrade"
],
```

Please note that `filament:upgrade` does not actually handle the update process, as Composer does that already. If you're upgrading manually without a `post-autoload-dump` hook, you can run the command yourself:

```bash
composer update

php artisan filament:upgrade
```

# Documentation for widgets. File: 02-stats-overview.md
---
title: Stats overview widgets
---

## Overview

Filament comes with a "stats overview" widget template, which you can use to display a number of different stats in a single widget, without needing to write a custom view.

Start by creating a widget with the command:

```bash
php artisan make:filament-widget StatsOverview --stats-overview
```

This command will create a new `StatsOverview.php` file. Open it, and return `Stat` instances from the `getStats()` method:

```php
<?php

namespace App\Filament\Widgets;

use Filament\Widgets\StatsOverviewWidget as BaseWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

class StatsOverview extends BaseWidget
{
    protected function getStats(): array
    {
        return [
            Stat::make('Unique views', '192.1k'),
            Stat::make('Bounce rate', '21%'),
            Stat::make('Average time on page', '3:12'),
        ];
    }
}
```

Now, check out your widget in the dashboard.

## Adding a description and icon to a stat

You may add a `description()` to provide additional information, along with a `descriptionIcon()`:

```php
use Filament\Widgets\StatsOverviewWidget\Stat;

protected function getStats(): array
{
    return [
        Stat::make('Unique views', '192.1k')
            ->description('32k increase')
            ->descriptionIcon('heroicon-m-arrow-trending-up'),
        Stat::make('Bounce rate', '21%')
            ->description('7% decrease')
            ->descriptionIcon('heroicon-m-arrow-trending-down'),
        Stat::make('Average time on page', '3:12')
            ->description('3% increase')
            ->descriptionIcon('heroicon-m-arrow-trending-up'),
    ];
}
```

The `descriptionIcon()` method also accepts a second parameter to put the icon before the description instead of after it:

```php
use Filament\Support\Enums\IconPosition;
use Filament\Widgets\StatsOverviewWidget\Stat;

Stat::make('Unique views', '192.1k')
    ->description('32k increase')
    ->descriptionIcon('heroicon-m-arrow-trending-up', IconPosition::Before)
```

## Changing the color of the stat

You may also give stats a `color()` (`danger`, `gray`, `info`, `primary`, `success` or `warning`):

```php
use Filament\Widgets\StatsOverviewWidget\Stat;

protected function getStats(): array
{
    return [
        Stat::make('Unique views', '192.1k')
            ->description('32k increase')
            ->descriptionIcon('heroicon-m-arrow-trending-up')
            ->color('success'),
        Stat::make('Bounce rate', '21%')
            ->description('7% increase')
            ->descriptionIcon('heroicon-m-arrow-trending-down')
            ->color('danger'),
        Stat::make('Average time on page', '3:12')
            ->description('3% increase')
            ->descriptionIcon('heroicon-m-arrow-trending-up')
            ->color('success'),
    ];
}
```

## Adding extra HTML attributes to a stat

You may also pass extra HTML attributes to stats using `extraAttributes()`:

```php
use Filament\Widgets\StatsOverviewWidget\Stat;

protected function getStats(): array
{
    return [
        Stat::make('Processed', '192.1k')
            ->color('success')
            ->extraAttributes([
                'class' => 'cursor-pointer',
                'wire:click' => "\$dispatch('setStatusFilter', { filter: 'processed' })",
            ]),
        // ...
    ];
}
```

In this example, we are deliberately escaping the `$` in `$dispatch()` since this needs to be passed directly to the HTML, it is not a PHP variable.

## Adding a chart to a stat

You may also add or chain a `chart()` to each stat to provide historical data. The `chart()` method accepts an array of data points to plot:

```php
use Filament\Widgets\StatsOverviewWidget\Stat;

protected function getStats(): array
{
    return [
        Stat::make('Unique views', '192.1k')
            ->description('32k increase')
            ->descriptionIcon('heroicon-m-arrow-trending-up')
            ->chart([7, 2, 10, 3, 15, 4, 17])
            ->color('success'),
        // ...
    ];
}
```

## Live updating stats (polling)

By default, stats overview widgets refresh their data every 5 seconds.

To customize this, you may override the `$pollingInterval` property on the class to a new interval:

```php
protected static ?string $pollingInterval = '10s';
```

Alternatively, you may disable polling altogether:

```php
protected static ?string $pollingInterval = null;
```

## Disabling lazy loading

By default, widgets are lazy-loaded. This means that they will only be loaded when they are visible on the page.

To disable this behavior, you may override the `$isLazy` property on the widget class:

```php
protected static bool $isLazy = false;
```

## Adding a heading and description

You may also add heading and description text above the widget by overriding the `$heading` and `$description` properties:

```php
protected ?string $heading = 'Analytics';

protected ?string $description = 'An overview of some analytics.';
```

If you need to dynamically generate the heading or description text, you can instead override the `getHeading()` and `getDescription()` methods:

```php
protected function getHeading(): ?string
{
    return 'Analytics';
}

protected function getDescription(): ?string
{
    return 'An overview of some analytics.';
}
```

# Documentation for widgets. File: 03-charts.md
---
title: Chart widgets
---

## Overview

Filament comes with many "chart" widget templates, which you can use to display real-time, interactive charts.

Start by creating a widget with the command:

```bash
php artisan make:filament-widget BlogPostsChart --chart
```

There is a single `ChartWidget` class that is used for all charts. The type of chart is set by the `getType()` method. In this example, that method returns the string `'line'`.

The `protected static ?string $heading` variable is used to set the heading that describes the chart. If you need to set the heading dynamically, you can override the `getHeading()` method.

The `getData()` method is used to return an array of datasets and labels. Each dataset is a labeled array of points to plot on the chart, and each label is a string. This structure is identical to the [Chart.js](https://www.chartjs.org/docs) library, which Filament uses to render charts. You may use the [Chart.js documentation](https://www.chartjs.org/docs) to fully understand the possibilities to return from `getData()`, based on the chart type.

```php
<?php

namespace App\Filament\Widgets;

use Filament\Widgets\ChartWidget;

class BlogPostsChart extends ChartWidget
{
    protected static ?string $heading = 'Blog Posts';

    protected function getData(): array
    {
        return [
            'datasets' => [
                [
                    'label' => 'Blog posts created',
                    'data' => [0, 10, 5, 2, 21, 32, 45, 74, 65, 45, 77, 89],
                ],
            ],
            'labels' => ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
        ];
    }

    protected function getType(): string
    {
        return 'line';
    }
}
```

Now, check out your widget in the dashboard.

## Available chart types

Below is a list of available chart widget classes which you may extend, and their corresponding [Chart.js](https://www.chartjs.org/docs) documentation page, for inspiration on what to return from `getData()`:

- Bar chart - [Chart.js documentation](https://www.chartjs.org/docs/latest/charts/bar)
- Bubble chart - [Chart.js documentation](https://www.chartjs.org/docs/latest/charts/bubble)
- Doughnut chart - [Chart.js documentation](https://www.chartjs.org/docs/latest/charts/doughnut)
- Line chart - [Chart.js documentation](https://www.chartjs.org/docs/latest/charts/line)
- Pie chart - [Chart.js documentation](https://www.chartjs.org/docs/latest/charts/doughnut.html#pie)
- Polar area chart - [Chart.js documentation](https://www.chartjs.org/docs/latest/charts/polar)
- Radar chart - [Chart.js documentation](https://www.chartjs.org/docs/latest/charts/radar)
- Scatter chart - [Chart.js documentation](https://www.chartjs.org/docs/latest/charts/scatter)

## Customizing the chart color

You can customize the color of the chart data by setting the `$color` property to either `danger`, `gray`, `info`, `primary`, `success` or `warning`:

```php
protected static string $color = 'info';
```

If you're looking to customize the color further, or use multiple colors across multiple datasets, you can still make use of Chart.js's [color options](https://www.chartjs.org/docs/latest/general/colors.html) in the data:

```php
protected function getData(): array
{
    return [
        'datasets' => [
            [
                'label' => 'Blog posts created',
                'data' => [0, 10, 5, 2, 21, 32, 45, 74, 65, 45, 77, 89],
                'backgroundColor' => '#36A2EB',
                'borderColor' => '#9BD0F5',
            ],
        ],
        'labels' => ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
    ];
}
```

## Generating chart data from an Eloquent model

To generate chart data from an Eloquent model, Filament recommends that you install the `flowframe/laravel-trend` package. You can view the [documentation](https://github.com/Flowframe/laravel-trend).

Here is an example of generating chart data from a model using the `laravel-trend` package:

```php
use Flowframe\Trend\Trend;
use Flowframe\Trend\TrendValue;

protected function getData(): array
{
    $data = Trend::model(BlogPost::class)
        ->between(
            start: now()->startOfYear(),
            end: now()->endOfYear(),
        )
        ->perMonth()
        ->count();

    return [
        'datasets' => [
            [
                'label' => 'Blog posts',
                'data' => $data->map(fn (TrendValue $value) => $value->aggregate),
            ],
        ],
        'labels' => $data->map(fn (TrendValue $value) => $value->date),
    ];
}
```

## Filtering chart data

You can set up chart filters to change the data shown on chart. Commonly, this is used to change the time period that chart data is rendered for.

To set a default filter value, set the `$filter` property:

```php
public ?string $filter = 'today';
```

Then, define the `getFilters()` method to return an array of values and labels for your filter:

```php
protected function getFilters(): ?array
{
    return [
        'today' => 'Today',
        'week' => 'Last week',
        'month' => 'Last month',
        'year' => 'This year',
    ];
}
```

You can use the active filter value within your `getData()` method:

```php
protected function getData(): array
{
    $activeFilter = $this->filter;

    // ...
}
```

## Live updating chart data (polling)

By default, chart widgets refresh their data every 5 seconds.

To customize this, you may override the `$pollingInterval` property on the class to a new interval:

```php
protected static ?string $pollingInterval = '10s';
```

Alternatively, you may disable polling altogether:

```php
protected static ?string $pollingInterval = null;
```

## Setting a maximum chart height

You may place a maximum height on the chart to ensure that it doesn't get too big, using the `$maxHeight` property:

```php
protected static ?string $maxHeight = '300px';
```

## Setting chart configuration options

You may specify an `$options` variable on the chart class to control the many configuration options that the Chart.js library provides. For instance, you could turn off the [legend](https://www.chartjs.org/docs/latest/configuration/legend.html) for a line chart:

```php
protected static ?array $options = [
    'plugins' => [
        'legend' => [
            'display' => false,
        ],
    ],
];
```

Alternatively, you can override the `getOptions()` method to return a dynamic array of options:

```php
protected function getOptions(): array
{
    return [
        'plugins' => [
            'legend' => [
                'display' => false,
            ],
        ],
    ];
}
```

These PHP arrays will get transformed into JSON objects when the chart is rendered. If you want to return raw JavaScript from this method instead, you can return a `RawJs` object. This is useful if you want to use a JavaScript callback function, for example:

```php
use Filament\Support\RawJs;

protected function getOptions(): RawJs
{
    return RawJs::make(<<<JS
        {
            scales: {
                y: {
                    ticks: {
                        callback: (value) => '€' + value,
                    },
                },
            },
        }
    JS);
}
```

## Adding a description

You may add a description, below the heading of the chart, using the `getDescription()` method:

```php
public function getDescription(): ?string
{
    return 'The number of blog posts published per month.';
}
```

## Disabling lazy loading

By default, widgets are lazy-loaded. This means that they will only be loaded when they are visible on the page.

To disable this behavior, you may override the `$isLazy` property on the widget class:

```php
protected static bool $isLazy = true;
```

## Using custom Chart.js plugins

Chart.js offers a powerful plugin system that allows you to extend its functionality and create custom chart behaviors. This guide details how to use them in a chart widget.

### Step 1: Install the plugin with NPM

To start with, install the plugin using NPM into your project. In this guide, we will install [`chartjs-plugin-datalabels`](https://chartjs-plugin-datalabels.netlify.app/guide/getting-started.html#installation):

```bash
npm install chartjs-plugin-datalabels --save-dev
```

### Step 2: Create a JavaScript file importing the plugin

Create a new JavaScript file where you will define your custom plugin. In this guide, we'll call it `filament-chart-js-plugins.js`. Import the plugin, and add it to the `window.filamentChartJsPlugins` array:

```javascript
import ChartDataLabels from 'chartjs-plugin-datalabels'

window.filamentChartJsPlugins ??= []
window.filamentChartJsPlugins.push(ChartDataLabels)
```

It's important to initialise the array if it has not been already, before pushing onto it. This ensures that mutliple JavaScript files (especially those from Filament plugins) that register Chart.js plugins do not overwrite each other, regardless of the order they are booted in.

You can push as many plugins to the `filamentChartJsPlugins` array as you would like to install, you do not need a separate file to import each plugin.

### Step 3: Compile the JavaScript file with Vite

Now, you need to build the JavaScript file with Vite, or your bundler of choice. Include the file in your Vite configuration (usually `vite.config.js`). For example:

```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/css/app.css',
                'resources/js/app.js',
                'resources/css/filament/admin/theme.css',
                'resources/js/filament-chart-js-plugins.js', // Include the new file in the `input` array so it is built
            ],
        }),
    ],
});
```

Build the file with `npm run build`.

### Step 4: Register the JavaScript file in Filament

Filament needs to know to include this JavaScript file when rendering chart widgets. You can do this in the `boot()` method of a service provider like `AppServiceProvider`:

```php
use Filament\Support\Assets\Js;
use Filament\Support\Facades\FilamentAsset;
use Illuminate\Support\Facades\Vite;

FilamentAsset::register([
    Js::make('chart-js-plugins', Vite::asset('resources/js/filament-chart-js-plugins.js'))->module(),
]);
```

You can find out more about [asset registration](../support/assets), and even [register assets for a specific panel](../panels/configuration#registering-assets-for-a-panel).

# Documentation for widgets. File: 04-tables.md
---
title: Table widgets
---

When using the [Panel Builder](../panels), you can use table widgets. These use the [table builder](../tables). You can find out how to create them [here](../panels/dashboard#table-widgets).

If you're not using the Panel Builder, there's no need to use a "widget" to render a table. You can simply [add a table to a Livewire component](../tables/adding-a-table-to-a-livewire-component), which does not provide any specific benefits over a widget.

# Documentation for widgets. File: 05-adding-a-widget-to-a-blade-view.md
---
title: Adding a widget to a Blade view
---

## Overview

Since widgets are Livewire components, you can easily render a widget in any Blade view using the `@livewire` directive:

```blade
<div>
    @livewire(\App\Livewire\Dashboard\PostsChart::class)
</div>
```

