# TD1 MRZ Overflow Format Validation

**What:** Fix TD1 (ID card) MRZ scanning for document numbers exceeding 9 characters using ICAO 9303 overflow format

**Status:** iOS implementation complete and tested. Android implementation complete. NFC functionality pending validation via TestFlight.

**Contact:** @armankolozyan (https://armankolozyan.com)

**How Self fits:** Enables Self to scan ID cards from any country using TD1 format with document numbers exceeding 9 characters (Belgium, Portugal, Spain, ...). Essential for NFC chip authentication (BAC keys) which requires the full untruncated document number.

## Context

Self enables users to prove their identity in a privacy-preserving manner by generating zero-knowledge proofs from their passport or ID card data. To read the ID card data, Self scans the Machine Readable Zone (MRZ) at the bottom of the document. The MRZ contains essential information like document number, date of birth, and expiry date encoded in a standardized format defined by ICAO 9303. The MRZ embeds check digits (single-digit numbers calculated from the data using a formula) throughout the encoded fields to detect OCR scanning errors.

Accurately scanning the MRZ is critical for two reasons. First, the identity data parsed from the MRZ is required for proof generation. Second, the NFC chip on modern IDs protects its data behind access control. With Basic Access Control (BAC), the reader and chip derive session keys from the exact MRZ values (document number, date of birth, and expiry date) and then establish encrypted, authenticated communication. If any MRZ field, such as the document number, is incorrect, the derived keys will not match, mutual authentication will fail, and the chip’s data cannot be read.

Travel documents use different MRZ formats depending on type. 
Passports use the TD3 format, while ID cards use the TD1 format. The TD1 format consists of three lines of exactly 30 characters each.

For document numbers exceeding 9 characters, ICAO 9303 Part 5 defines a special "overflow mechanism":

> "If Document Number on a TD-1 has more than 9 characters. The nine (9) principal (most significant) characters shall be recorded in Data Element 04. Data Element 05 shall be coded with the filler character (<). The remaining characters of the document number shall be recorded at the beginning of Data Element 12 (Optional Data) followed by the Document Number Check digit and a filler character (<)."
>
> Source: [LDS Technical Report 2004](https://is.muni.cz/el/1433/podzim2007/PV181/um/ep/LDS-technical_report_2004.pdf)

> "if the document number has more than 9 characters, the 9 principal characters shall be shown in the MRZ in character positions 6 to 14. They shall be followed by a filler character instead of a check digit to indicate a truncated number. The remaining characters of the document number shall be shown at the beginning of the field reserved for optional data elements (character positions 16 to 30 of the upper machine readable line) followed by a check digit and a filler character."
>
> Source: [ICAO Doc 9303 Part 5](https://www.icao.int/sites/default/files/publications/DocSeries/9303_p5_cons_en.pdf)

This case was discovered when testing Self with a Belgian ID card that failed to scan properly. The MRZ scanner would detect the text but validation consistently failed. Upon investigation on an iOS device, it was determined that the existing MRZ parsing library (QKMRZParser) does not correctly handle the TD1 overflow format defined in the ICAO 9303 standard.

This overflow format is used by ID cards worldwide when document numbers exceed 9 characters, affecting multiple countries including Belgium, Portugal, Spain, and others. Without proper overflow handling, these ID cards cannot be scanned in Self, and even if scanned, the truncated document number would cause NFC chip authentication to fail.

## Problem

Three critical issues were identified with the existing MRZ validation logic:

### 1. Parser Hard-Coded Indices Breaking Initial Attempted Fix

[An initial fix](https://github.com/selfxyz/self/pull/1061) was attempted that detected Belgian IDs by country code (`countryCode == "BEL"`), then manually extracted overflow digits from the optional data field (positions 16+) and concatenated them with the principal part to reconstruct the full document number. This approach seemed logical: detect Belgian IDs, manually handle the overflow format, and pass the corrected document number to the rest of the system.
However, this fix was broken because the underlying QKMRZParser (iOS) uses hard-coded field indices that don't account for the overflow mechanism. The parser extracts the document number from positions 6-14 only and validates it with the check digit at position 15, completely ignoring the overflow digits at positions 16+.

**Result**: The custom overflow extraction logic had no effect.

### 2. Country-Specific Implementation

The initial fix was [hard-coded to only work with Belgian IDs](https://github.com/selfxyz/self/blob/ee3a546469355cd12f30f48b103c157ff51d8f97/app/ios/LiveMRZScannerView.swift#L211) (`countryCode == "BEL"`). However, the TD1 overflow format is part of the ICAO 9303 international standard and is used by many countries.

**Result**: ID cards from other countries with long document numbers would still fail validation.

### 3. Stripping First 3 Characters for No Specific Reason

The initial implementation was [stripping](https://github.com/selfxyz/self/blob/ee3a546469355cd12f30f48b103c157ff51d8f97/app/ios/LiveMRZScannerView.swift#L113) the first 3 characters from the document number. This wasn't based on any ICAO specification.

**Result**: NFC authentication would fail because the modified document number wouldn't match the one stored in the chip.

## Solution

### Changes in `LiveMRZScannerView.swift`

#### 1. Added ICAO 9303 Check Digit Calculator
```swift
private func calculateMRZCheckDigit(_ input: String) -> Int
```
- Implements ICAO 9303 standard algorithm
- Maps: `0-9` → 0-9, `A-Z` → 10-35, `<` → 0
- Returns checksum mod 10

#### 2. Added TD1 Overflow Format Handler
```swift
private func extractAndValidateTD1DocumentNumber(line1: String) -> (documentNumber: String, isValid: Bool)?
```
- Detects overflow format when position 15 = `<`
- Extracts principal part (positions 6-14)
- Scans overflow digits (positions 16+)
- Last overflow character is the check digit
- Constructs full document number and validates check digit manually
- Handles both standard and overflow formats

#### 3. Added TD1 Document Processor
```swift
private func processTD1DocumentWithOverflow(result: String, parser: QKMRZParser) -> QKMRZResult?
```
- Uses manual validation for document number in case of overflow, instead of relying on QKMRZParser
- Returns `nil` if manual check digit validation fails
- Uses QKMRZParser only for extracting other fields (name, dates, nationality)
- Stores validated full document number in `overrideDocumentNumber` state variable

#### 4. Updated Document Number Mapping
```swift
private func mapVisionResultToDictionary(_ result: QKMRZResult) -> [String: Any]
```
- Uses `overrideDocumentNumber` if available (for TD1 overflow cases)
- Falls back to parser's document number for standard cases

## TODO

1. **Test NFC authentication on iOS** - Validate that the full untruncated document number works correctly for NFC chip authentication (BAC key derivation). Justin is creating a TestFlight build for physical device testing.

2. **Check if same issue exists on Android** - Test Android MRZ scanning with ID cards that use TD1 overflow format to verify if the same limitation exists.

3. **Implement Android solution if needed** - If Android has the same issue, implement the TD1 overflow validation following the same approach as iOS (manual check digit validation bypassing the parser's hard-coded indices).

4. **Consider updating the parser library** - Alternatively, investigate whether QKMRZParser (iOS) and jmrtd (Android) can be updated or forked to natively support TD1 overflow format, which would be a more sustainable long-term solution than the current workaround.

## Updates

- **Week 1:** Not in the residency yet :(.
- **Week 2:** Identified and investigated issue with Belgian IDs.
- **Week 3:** Implemented fix for Belgian IDs, then generalized the solution to support all countries with the same issue (suggested by Marek).

## Contact

For questions or issues related to this implementation, please contact: **@armankolozyan** (https://armankolozyan.com)
