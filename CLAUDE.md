# Implementation Plan: OSM POS Import Feature

## Overview
This document outlines the implementation of the OpenStreetMap (OSM) import feature for the CampusCoffee application. The feature allows API users to create Point of Sale (POS) entries by importing data from OpenStreetMap using OSM node IDs.

## Current State
- CHANGELOG.md documents the feature requirements
- PosService interface has `importFromOsmNode` method defined
- PosController has the POST endpoint `/api/pos/import/osm/{nodeId}` 
- Stub implementations exist in OsmDataServiceImpl and PosServiceImpl
- OsmNode model only contains nodeId, needs extension with OSM tag fields

## Implementation Tasks

### 1. Extend OsmNode Domain Model
**File**: `domain/src/main/java/de/seuhd/campuscoffee/domain/model/OsmNode.java`
- Add OSM tag fields relevant to POS (name, street, housenumber, postcode, city, etc.)
- Use @Builder for object construction
- Annotate with @NonNull for required fields, @Nullable for optional fields

**Required OSM Fields for POS**:
- `name` - Coffee shop name
- `addr:street` - Street name
- `addr:housenumber` - House number
- `addr:postcode` - Postal code
- `addr:city` - City name
- `amenity` - Type (typically "cafe")
- `description` - Optional description
- `cuisine` - Type of cuisine/cafe
- `internet_access:fee` - Internet access info
- `indoor_seating` - Boolean for indoor seating
- `outdoor_seating` - Boolean for outdoor seating
- `opening_hours` - Operating hours
- `phone` - Contact number
- `website` - Website URL
- `smoking` - Smoking policy
- `diet:vegan` - Vegan options available
- `diet:vegetarian` - Vegetarian options available
- `name:de` - German name
- `name:en` - English name

### 2. Enhance OsmDataServiceImpl
**File**: `data/src/main/java/de/seuhd/campuscoffee/data/impl/OsmDataServiceImpl.java`
- Implement HTTP client to fetch from OSM API: `https://www.openstreetmap.org/api/0.6/node/{id}?format=json`
- Parse XML/JSON response from OSM API
- Extract relevant tags and map to OsmNode fields
- Handle network errors, invalid responses, missing nodes
- Use RestTemplate or WebClient (Spring) for HTTP communication

### 3. Implement OsmNode to POS Conversion
**File**: `domain/src/main/java/de/seuhd/campuscoffee/domain/impl/PosServiceImpl.java`
- Replace stub `convertOsmNodeToPos` method with real mapping logic
- Handle missing/optional fields with sensible defaults
- Map `amenity` tag to PosType (extract type information)
- Determine campus location (hardcoded for now, or from configuration)
- Validate all required fields are present, throw OsmNodeMissingFieldsException if not
- Handle special characters in names and addresses

**Mapping Strategy**:
- POS name: Use `name` or `name:en` or `name:de` (priority order)
- Description: Use `description` field if available, else build from `cuisine` + `amenity`
- Type: Parse from `amenity` tag (e.g., "cafe" → PosType.CAFE)
- Campus: Default to ALTSTADT (can be enhanced with geolocation in future)
- Address fields: Map from `addr:*` OSM tags

### 4. Add Exception Handling
**File**: `domain/src/main/java/de/seuhd/campuscoffee/domain/exceptions/`
- Verify OsmNodeMissingFieldsException exists and is properly used
- Ensure all exceptions in API controller are properly caught and return HTTP error responses

### 5. Update API Controller Error Handling
**File**: `api/src/main/java/de/seuhd/campuscoffee/api/controller/PosController.java`
- Add @ExceptionHandler methods or ensure GlobalExceptionHandler catches:
  - OsmNodeNotFoundException → HTTP 404
  - OsmNodeMissingFieldsException → HTTP 400
  - DuplicatePosNameException → HTTP 409
- Return proper error response with meaningful messages

### 6. Create/Update Integration Tests
**File**: `application/src/test/java/de/seuhd/campuscoffee/systest/PosSystemTests.java`
- Test the complete import flow with a real OSM node ID
- Mock HTTP calls to OSM API for reliable testing
- Test error cases (node not found, missing fields, duplicate names)
- Verify returned POS has correct fields populated

### 7. Update README with Example
**File**: `README.md`
- Verify the curl example for OSM import is present and correct
- Already documented: `curl --request POST http://localhost:8080/api/pos/import/osm/5589879349`

## XML Output Consideration
The requirement mentions "XML file of every entry/node" should showcase the POS features. This likely refers to:
- Database persistence (already handled by Spring Data JPA)
- API responses return JSON (standard REST practice)
- XML export might be a future enhancement, or already managed by external systems
- Focus on ensuring all POS fields are properly persisted and retrievable

## Implementation Order
1. Extend OsmNode model with all required fields
2. Implement OsmDataServiceImpl HTTP client logic
3. Implement POS conversion logic in PosServiceImpl
4. Add proper exception handling
5. Create comprehensive tests
6. Verify README documentation is complete

## Output Format
Implementation code will be provided as JSON with file paths and code content for easy application.
