# PLSS data packs

Prepared Public Land Survey System data for the takwerx **PLSS Grid** plugin for
ATAK. The plugin ships with no map data; it reads `manifest.json` from a release
here and downloads the state packs the operator selects.

The packs are also usable on their own — they are ordinary SQLite files.

## Source and provenance

Derived from the **BLM National PLSS CadNSDI**, which is the authoritative
survey record for the Public Land Survey System:

    https://gis.blm.gov/arcgis/rest/services/Cadastral/BLM_Natl_PLSS_CadNSDI/MapServer

That is a work of the United States government and is in the public domain.
This repository redistributes a re-encoded subset of it. Each release records
the date the source was retrieved.

Feature counts are verified against the service at build time. The current
release holds **85,983 townships** and **2,773,678 sections** across 30 states —
three sections short of the service's 2,773,681, all three being records BLM
cannot place or draw (two carry no state in `PLSSID`, one has no geometry).

## Coverage

PLSS covers 30 states. It is absent by design where the rectangular survey never
ran — most of the eastern seaboard, Texas, and land held under Spanish and
Mexican grants, which is why there is no PLSS beneath much of coastal and
southern California. Gaps in those areas are correct.

## Format

One SQLite file per state, `plss_<ST>.sqlite`:

| table | contents |
|---|---|
| `township` | id, plssid, label (`T19S-R25E`), meridian, geometry |
| `section` | id, plssid, divid, divno, label, geometry |
| `township_idx`, `section_idx` | R-tree bounding boxes |
| `meta` | schema, state, scale, feature counts |

Geometry is quantized to 1e-6 degrees (~0.11 m) and delta-varint encoded, about
63 bytes per section. Each blob begins with its origin as two zigzag varints,
then a ring count, then per ring a vertex count and zigzag-varint deltas.

Note that the R-tree index requires a SQLite built with `SQLITE_ENABLE_RTREE`.
Android's bundled SQLite is not; the plugin reaches the index through ATAK's own
SQLite instead.

## manifest.json

    {
      "schema": 1,
      "source": "BLM CadNSDI PLSS",
      "sourceDate": "2026-08-19",
      "packs": [
        { "state": "CA", "name": "California", "url": "plss_CA.sqlite",
          "bytes": 28770304, "sha256": "...", "townships": 4714,
          "sections": 141935, "schema": 1 }
      ]
    }

Pack `url` values are relative to the manifest, so the whole set can be moved
between hosts without editing it.

## Building these

The pipeline lives in the plugin repository under `plugins/PLSS/tools/` —
`fetch_blm.py`, `pack_plss.py`, `make_manifest.py`. Python 3, standard library
only.
