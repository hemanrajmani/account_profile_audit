# 🔍 Account Profile Audit – Audit Logic Documentation

## 📌 Overview

This document explains the audit validation rules, business logic, and Excel formulas used in the **Account Profile Audit** project.

The audit framework is designed to identify:
- Data quality issues
- Formatting inconsistencies
- Invalid address structures
- Incorrect country/state codes
- Improper tax ID formats
- Duplicate records
- Special characters
- Standardization issues

The objective is to improve account master data quality and support reliable operational reporting.

---

# 🛠 Audit Validation Rules

---

# 1. Unique Record Validation

## Audit Field
`Unique_Record_Flag`

## Purpose
Filters unique records to avoid duplicate rows.

## Formula
```excel
=IF(G2=G1,"","Unique Record")
```

## Business Logic
Flags only unique records for audit processing as the CRM exports the change history of the records in multiple rows.

---

# 2. Account Upper Case Validation

## Audit Field
`Account_Upper Case_Flag`

## Purpose
Checks whether the Account Name is entered completely in uppercase.

## Formula
```excel
=IF(F2="","",IF(EXACT(F2,(UPPER(F2))),"Account Upper Case",""))
```

## Example
| Invalid |
|---|
| ABC CORPORATION |

---

# 3. Account Lower Case Validation

## Audit Field
`Account_Lower Case__Flag`

## Purpose
Checks whether the Account Name is entered completely in lowercase.

## Formula
```excel
=IF(F2="","",IF(EXACT(F2,(LOWER(F2))),"Account Lower Case",""))
```

## Example
| Invalid |
|---|
| abc corporation |

---

# 4. Billing Street Proper Case Validation

## Audit Field
`Billing Street_ProperCase_Flag`

## Purpose
Checks whether Billing Street values follow proper casing standards.

## Formula
```excel
=IF(EXACT(H2,PROPER(H2)),"","Billing Street - casing")
```

## Example
| Invalid | Valid |
|---|---|
| MAIN STREET | Main Street |

---

# 5. Billing City Proper Case Validation

## Audit Field
`Billing City_ProperCase_Flag`

## Purpose
Checks whether Billing City values follow proper case formatting.

## Formula
```excel
=IF(EXACT(I2,PROPER(I2)),"","City - casing")
```

---

# 6. Billing City Saint Validation

## Audit Field
`Billing_City_Saint_Flag`

## Purpose
Checks whether “Saint” abbreviations (`St` or `St.`) are used correctly in city names.

## Formula
```excel
=IF(OR(ISNUMBER(SEARCH("St.",I2)),ISNUMBER(SEARCH("St ",I2))),"Check St in city field",IF(RIGHT(I2,1)=".","Dot found in city field",""))
```

## Example
| Potential Issues |
|---|
| St Louis |
| Paris. |

---

# 7. Billing City Special Character Validation

## Audit Field
`Billing_City_Special_Char_Flag`

## Purpose
Detects invalid symbols and special characters in city names.

## Validation Logic
Uses `SEARCH` and `SUMPRODUCT` functions to detect predefined special characters.

## Output
```text
Special Characters Found in City
```

---

# 8. Account Special Character Validation

## Audit Field
`Account_Special_Char_Flag`

## Purpose
Detects invalid special characters in Account Name fields.

## Output
```text
Special Characters Found in Account field
```

## Business Impact
Special characters may cause:
- CRM inconsistencies
- Integration failures
- Invalid reporting outputs

---

# 9. Address Directional Validation

## Audit Field
`Address1_Directionals_Flag`

## Purpose
Checks whether directional indicators are entered using standardized abbreviations.

## Supported Directionals
```text
N, S, E, W, NE, NW, SE, SW
```

## Validation Logic
The formula validates:
- Directional abbreviations
- Street suffix handling
- Invalid directional words
- Formatting inconsistencies

## Output
```text
Directional Check
```

---

# 10. State Code Validation

## Audit Field
`State_Code_Flag`

## Purpose
Checks whether state/province codes are valid based on country standards.

## Supported Countries
- United States (US)
- Canada (CA)
- Brazil (BR)
- Australia (AU)

## Validation Logic
Uses `VLOOKUP` against the `Country Codes` reference table.

## Possible Outputs
```text
US State Code Error
Canada State Code Error
Brazil State Code Error
Australia State Code Error
```

---

# 11. State Upper Case Validation

## Audit Field
`State_Upper_Case_Flag`

## Purpose
Checks whether state codes follow proper casing rules.

## Validation Rules
| Country | Expected Format |
|---|---|
| US / CA / BR / AU | Upper Case |
| Others | Proper Case |

## Formula
```excel
=IF(OR(L2="US",L2="CA",L2="BR",L2="AU"),IF((EXACT(UPPER(J2),J2)),"","Country Code Casing"),IF(EXACT(J2,PROPER(J2)),"","Billing State casing"))
```

---

# 12. Address Validation

## Audit Field
`Address_Flag`

## Purpose
Checks whether City, State, Postal Code, or Country values are incorrectly included in the Address field.

## Validation Logic
The formula searches for:
- City values
- Postal codes
- State codes
- Country codes

inside the Address field.

## Output
```text
City/Postal Code/Country Found in Address
```

---

# 13. Incorrect Tax ID Field Validation

## Audit Field
`Incorrect_TaxID_Field_Flag`

## Purpose
Checks whether PO numbers are incorrectly entered inside the Tax ID field.

## Formula
```excel
=IF(ISNUMBER(SEARCH("PO",N2)),"PO# Entered in Tax ID Field","")
```

---

# 14. Incorrect PO Number Validation

## Audit Field
`Incorrect_PO#_Flag`

## Purpose
Checks whether PO Numbers contain invalid prefixes.

## Validation Logic
Uses lookup validation against predefined PO standards in the Country Codes reference sheet.

## Output
```text
PO Number may be incorrect
```

---

# 15. Tax ID / VAT Validation

## Audit Field
`Tax ID_VAT_Validation`

## Purpose
Checks whether VAT/Tax ID values contain valid country codes and formatting.

## Validation Checks
- Country code validation
- Space validation after country code

## Formula
```excel
=IF(N2="","",IF(EXACT(L2,LEFT(N2,2)),IF(EXACT(MID(N2,3,1)," "),"Check the Space inbetween County code & VAT",""),"Check the VAT Country Code Format"))
```

## Possible Outputs
```text
Check the Space inbetween County code & VAT
Check the VAT Country Code Format
```

---

# 16. Billing Contact Proper Case Validation

## Audit Field
`Contact_ProperCase_Flag`

## Purpose
Checks whether Billing Contact Names follow proper casing standards.

## Special Handling
Supports apostrophe-based names such as:
```text
O'Connor
D'Souza
```

## Output
```text
Please check Billing Contact casing
```

---

# 17. US Territories Validation

## Audit Field
`US_Territories_Flag`

## Purpose
Flags US Territory country/state codes for validation review.

## Supported Territory Codes
```text
AS, FM, GU, MH, MP, PW, PR, VI, UM
```

## Output
```text
US Territories Check
```

---

# 18. Country Code Validation

## Audit Field
`Country_Code_Flag`

## Purpose
Checks whether the Country Code exists in the reference table.

## Formula
```excel
=IF(G2="","",IFNA(IFS(L2=VLOOKUP(L2,'Country Codes'!$M:$M,1,0)=TRUE,""),"Please check the country code"))
```

## Output
```text
Please check the country code
```

---

# 19. Consolidated Error Remarks

## Audit Field
`Consolidate_Error_Remarks`

## Purpose
Combines all validation errors into a consolidated audit remarks field.

## Business Logic
Concatenates all triggered audit flags into a semicolon-separated summary.

## Example Output
```text
Account Upper Case;
Billing Street - casing;
Special Characters Found in City;
```

## Business Value
Provides auditors with a centralized issue summary for faster review and remediation.

---

# 20. Audited Status

## Audit Field
`Audited_Status`

## Purpose
Allows auditors to manually update the final audit review status.

## Example Status Values
- Reviewed
- Completed

---

# 📊 Dashboard Integration

The audit results are integrated into an interactive dashboard using:
- Charts
- KPI Cards

## Dashboard Metrics
- Total Records
- Records with Errors
- Error Distribution
- Most Common Audit Issues
- Clean vs Review Required Records

---

# 🚀 Future Enhancements

Potential future improvements include:

- VBA macro automation for dynamic audit execution
- One-click audit processing using Excel buttons/macros
- Automatic formula population based on dataset size
- Automated Pivot Table & Dashboard refresh
- Automated data quality scoring system

---

# 📌 Conclusion

This project demonstrates a real-world Excel-based Account Profile Audit framework designed to improve:
- Data governance
- CRM data quality
- Operational consistency
- Reporting reliability
- Standardization compliance

The solution combines business audit logic, Excel validation techniques, and dashboard reporting into a reusable data quality audit framework.
