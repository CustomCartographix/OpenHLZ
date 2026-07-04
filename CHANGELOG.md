# Changelog

All notable changes to OpenHLZ are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.0.1] - 2026-07-04

### Fixed

- **Mosaic/clip memory blowup on large AOIs.** `mosaicAndClipRasters` no longer
  uses `gdal:merge`, which materialised the entire output mosaic in memory and
  could exhaust RAM on wide search areas. It now reprojects, mosaics, and clips
  the tiles to the AOI in a single streaming `osgeo.gdal.Warp` pass that reads
  block-by-block, natively accepts the tiles in their different UTM CRSs,
  applies the AOI as a cutline, preserves the source colour table, and writes an
  LZW-compressed, tiled GeoTIFF. Nearest-neighbour resampling is used so
  categorical land-cover class values are never blended into invalid classes.
- **Malformed HLS raster-calculator expression.** The final "land cover x slope"
  expression was missing a closing quote (`"slope_reclass@1`), which could cause
  the last raster step to fail; it is now `"lc_reclass@1" * "slope_reclass@1"`.
- **Silent download failures.** Network errors are now raised as a
  `QgsProcessingException` instead of writing an HTML error page into a `.tif`
  and failing cryptically later in GDAL.
- **Leaked file handles.** All downloads now write through a `with open(...)`
  context manager, and the redundant open/close-immediately calls were removed.

### Changed

- **Land cover source updated to the 2025 collection.** Tiles are now pulled from
  Esri's `lc2025/{zone}_20250101-20251231.tif` Living Atlas layer.
- **`qgisMinimumVersion` raised to 3.34.** This backs the use of
  `TemporaryDirectory(delete=True, ignore_cleanup_errors=True)` (which requires
  Python 3.10/3.12) and the `writeAsVectorFormatV3` API.
- **Replaced deprecated `QgsVectorFileWriter.writeAsVectorFormat`** with
  `writeAsVectorFormatV3` throughout, behind a shared `writeToShapefile` helper.
- **Removed `chdir()`.** All bundled resource and style paths are now built as
  absolute paths from `os.path.dirname(__file__)`, so the plugin no longer
  mutates the QGIS process's global working directory.
- **Output styling now uses processing post-processors.** The HLS raster and HLZ
  points styles are applied via `QgsProcessingLayerPostProcessorInterface` to the
  layers the Processing framework loads on completion, instead of manually
  adding duplicate layers to the project.
- **Progress/cancellation threaded into downloads and the warp step** via the
  algorithm `feedback` object.

### Refactored

- **Deduplicated the automatic-download algorithms.** The Lat/Lng, Point, and AOI
  algorithms now share a single `_BaseDownloadHLZAlgorithm` that owns the common
  parameters and the full download -> mosaic/clip -> HLS -> HLZ pipeline; each
  subclass only implements how it builds the AOI. The HLS/HLZ tail is shared with
  the "Existing Data" algorithm through `_BaseHLZAlgorithm`.
- Bumped `OpenHLZProvider.longName()` to `OpenHLZ v2.0.1`.

## [2.0.0] - 2026-01-31

- Removed experimental flag.
- Changed data source options to fix bugs.
- Added the ability to use pre-downloaded data for analysis.
