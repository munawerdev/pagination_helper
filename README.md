# Flutter Pagination Helper

A lightweight and reusable Flutter package for implementing pagination with minimal boilerplate. Perfect for apps that need to load data incrementally from APIs.

## Features

- **PaginatedListView**: Automatic infinite scrolling list with pull-to-refresh
- **PaginatedGridView**: Grid layout with pagination support
- **PaginationMixin**: Powerful mixin for Cubit/Bloc with zero boilerplate
- **Flexible**: Works with any API structure (offset-based, page-based, cursor-based)
- **Type-safe**: Fully generic implementation
- **Customizable**: Loading indicators, empty states, thresholds, and more

## Installation

Add this to your `pubspec.yaml`:

```yaml
dependencies:
  flutter_pagination_helper: ^1.0.0
  flutter_bloc: ^8.1.3
  flutter_screenutil: ^5.9.0
```

## Quick Start

### 1. Using PaginatedListView

```dart
import 'package:flutter_pagination_helper/flutter_pagination_helper.dart';

PaginatedListView<Product>(
  items: state.products,
  isLoadingMore: state.isLoadingMore,
  onRefresh: () => cubit.refresh(),
  onLoadMore: () => cubit.loadMore(),
  itemBuilder: (context, product, index) {
    return ListTile(
      title: Text(product.name),
      subtitle: Text(product.price),
    );
  },
  emptyWidget: const Center(
    child: Text('No products found'),
  ),
)
```

### 2. Using PaginationMixin in Cubit

```dart
class ProductCubit extends Cubit<ProductState> with PaginationMixin {
  ProductCubit() : super(ProductState.initial());

  Future<void> loadMore() async {
    await loadMoreData<ProductData>(
      fetchData: (offset, limit) async {
        final response = await api.getProducts(
          skip: offset,
          limit: limit,
        );
        return ApiResponse.completed(response);
      },
      mergeData: (current, newData) => current.copyWith(
        products: [...current.products, ...newData.products],
      ),
      getCurrentCount: (data) => data.products.length,
      getTotalCount: (data) => data.total,
      limit: 20,
    );
  }

  Future<void> refresh() async {
    emit(state.copyWith(
      response: ApiResponse.completed(ProductData.empty()),
      isLoadingMore: false,
    ));
    await loadMore();
  }
}
```

### 3. State Setup

```dart
class ProductState {
  final ApiResponse<ProductData> response;
  final bool isLoadingMore;

  ProductState({
    required this.response,
    this.isLoadingMore = false,
  });

  factory ProductState.initial() {
    return ProductState(
      response: ApiResponse.completed(ProductData.empty()),
    );
  }

  ProductState copyWith({
    ApiResponse<ProductData>? response,
    bool? isLoadingMore,
  }) {
    return ProductState(
      response: response ?? this.response,
      isLoadingMore: isLoadingMore ?? this.isLoadingMore,
    );
  }
}

class ProductData {
  final List<Product> products;
  final int total;

  ProductData({required this.products, required this.total});

  factory ProductData.empty() => ProductData(products: [], total: 0);

  ProductData copyWith({List<Product>? products, int? total}) {
    return ProductData(
      products: products ?? this.products,
      total: total ?? this.total,
    );
  }
}
```

## Advanced Usage

### Page-Based Pagination

If your API uses page numbers instead of offsets:

```dart
await loadMoreWithPage<ProductData>(
  fetchData: (page, limit) async {
    final response = await api.getProducts(page: page, limit: limit);
    return ApiResponse.completed(response);
  },
  mergeData: (current, newData) => current.copyWith(
    products: [...current.products, ...newData.products],
  ),
  getCurrentCount: (data) => data.products.length,
  getTotalCount: (data) => data.total,
);
```

### Custom Loading Widget

```dart
PaginatedListView<Product>(
  items: products,
  isLoadingMore: isLoadingMore,
  onLoadMore: () => cubit.loadMore(),
  itemBuilder: (context, product, index) => ProductCard(product),
  loadingWidget: const Padding(
    padding: EdgeInsets.all(16),
    child: Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        CircularProgressIndicator(),
        SizedBox(width: 16),
        Text('Loading more products...'),
      ],
    ),
  ),
)
```

### Grid View with Pagination

```dart
PaginatedGridView<Product>(
  items: products,
  isLoadingMore: isLoadingMore,
  crossAxisCount: 2,
  crossAxisSpacing: 12,
  mainAxisSpacing: 12,
  childAspectRatio: 0.7,
  onLoadMore: () => cubit.loadMore(),
  itemBuilder: (context, product, index) {
    return ProductGridCard(product: product);
  },
)
```

### Custom Load More Threshold

```dart
PaginatedListView<Product>(
  items: products,
  isLoadingMore: isLoadingMore,
  loadMoreThreshold: 500.0, // Trigger 500px before bottom
  onLoadMore: () => cubit.loadMore(),
  itemBuilder: (context, product, index) => ProductCard(product),
)
```

### With Separators

```dart
PaginatedListView<Product>(
  items: products,
  isLoadingMore: isLoadingMore,
  onLoadMore: () => cubit.loadMore(),
  itemBuilder: (context, product, index) => ProductCard(product),
  separatorBuilder: (context, index) => const Divider(),
)
```

## API Reference

### PaginatedListView

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| items | List\<T\> | Yes | List of items to display |
| isLoadingMore | bool | Yes | Whether currently loading more items |
| itemBuilder | Widget Function | Yes | Builder for individual items |
| onLoadMore | VoidCallback | Yes | Called when more items needed |
| onRefresh | Future\<void\> Function()? | No | Pull-to-refresh callback |
| loadingWidget | Widget? | No | Custom loading indicator |
| emptyWidget | Widget? | No | Widget shown when list is empty |
| loadMoreThreshold | double | No | Distance from bottom to trigger load (default: 200) |
| separatorBuilder | Widget Function? | No | Builder for item separators |
| enableRefresh | bool | No | Enable pull-to-refresh (default: true) |

### PaginatedGridView

Includes all parameters from PaginatedListView plus:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| crossAxisCount | int | Yes | Number of columns |
| childAspectRatio | double | No | Width/height ratio (default: 1.0) |
| crossAxisSpacing | double | No | Horizontal spacing (default: 0) |
| mainAxisSpacing | double | No | Vertical spacing (default: 0) |

### PaginationMixin

#### loadMoreData\<TData\>

| Parameter | Type | Description |
|-----------|------|-------------|
| fetchData | Future\<ApiResponse\<TData\>\> Function(int, int) | Fetch function receiving (offset, limit) |
| mergeData | TData Function(TData, TData) | Function to merge current and new data |
| getCurrentCount | int Function(TData) | Get current item count |
| getTotalCount | int Function(TData) | Get total available items |
| limit | int | Items per page (default: 10) |

## Requirements

- Flutter: >=3.0.0
- Dart: >=3.0.0 <4.0.0

## License

MIT License

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## Author

Your Name

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for release notes.