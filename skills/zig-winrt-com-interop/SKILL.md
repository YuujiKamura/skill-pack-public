---
name: zig-winrt-com-interop
description: "WinUI3, WinRT, Zig, COM, vtable, ghostty. (Pure Zig WinRT Architecture + ABI Hidden Pointers) Use when user mentions: WinUI3, WinRT, Zig, COM, vtable, ghostty, XAML."
---

# Miscellaneous

Patterns: 1

## 1. Pure Zig WinRT Architecture

Building WinUI 3 applications in Zig without C++ dependencies requires manual mapping of WinRT COM vtables and ABI management.

### Steps

1. Direct mapping of WinRT COM vtables (using IUnknown and IInspectable) in Zig allows for a C++-less implementation, reducing build complexity and toolchain bloat.
2. Strict binary layout compatibility is the primary risk; every method in the vtable must be accounted for in the correct order to avoid segmentation faults.
3. This 'Direct Mapping' approach is preferred over C wrappers when the goal is a portable, single-language codebase, but it requires manual maintenance of the COM interface definitions.

### Examples

```zig
const IInspectable = struct {
 lpVtbl: *const VTable,
 pub const VTable = struct {
 QueryInterface: *const anyopaque,
 AddRef: *const anyopaque,
 Release: *const anyopaque,
 GetIids: *const anyopaque,
 GetRuntimeClassName: *const anyopaque,
 GetTrustLevel: *const anyopaque,
 };
};
```

## 2. WinRT x64 ABI Hidden Pointers for Large Structs

x64 Windows ABI で 8 バイト超の構造体を WinRT メソッドに渡す際の呼び出し規約対応。

### Steps

1. x64 Windows ABI では、8 バイト超の構造体（WinRT の 16 バイト `TypeName` など）は呼び出し元が提供する hidden pointer 経由で渡される。
2. COM インタフェースを手動実装・呼び出しする場合（例: `IXamlMetadataProvider`）、これらの構造体を値渡しできない。`@ptrCast(&struct_instance)` を使って ABI の期待するレジスタ/スタック状態に合わせる必要がある。
3. この hidden pointer 規約を無視するとアラインメントずれが発生し、委譲先関数突入時に即座にクラッシュする。

### Examples

```zig
// TypeName is 16 bytes; pass by pointer to match x64 ABI expectation
_ = provider.lpVtbl.GetXamlType(provider, @ptrCast(&type_name), &result);
```



