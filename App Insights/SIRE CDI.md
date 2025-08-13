# Maritime Inspection Data Processing System

## Overview

Our Maritime Inspection Data Processing System provides comprehensive analysis and reporting of vessel inspection data from two primary sources: **SIRE (Ship Inspection Report Programme)** and **CDI (Chemical Distribution Institute)** inspections. The system processes inspection records obtained through API calls from OCIMF and CDI websites, which are then stored in ERP databases and processed to generate detailed reports, interactive charts, and data tables for maritime industry stakeholders.

## System Capabilities

### SIRE Inspection Processing
- **Fleet-wide Analysis**: Processes inspection data across entire vessel fleets
- **Individual Vessel Reports**: Detailed inspection history for specific vessels
- **Observation Analytics**: Statistical analysis of inspection observations by chapter
- **Validity Tracking**: Automatic calculation of inspection validity periods and due dates

### CDI Inspection Processing
- **Chemical Carrier Focus**: Specialized processing for chemical tanker inspections
- **Observation Management**: Detailed tracking of CDI-specific inspection observations
- **Compliance Monitoring**: Validity period tracking for chemical distribution requirements

## Data Flow Architecture

### 1. Data Collection
The system follows a multi-stage data collection process:

#### External API Data Sources
- **OCIMF Website API**: SIRE inspection data retrieved through API calls
- **CDI Website API**: Chemical Distribution Institute inspection data via API calls

#### Internal Data Processing
API-sourced data is stored and categorized in ERP databases:
- **SIRE Reports**: Published inspection reports with detailed observations
- **SIRE Pending**: Inspections awaiting final publication
- **SIRE Unprocessed**: Raw inspection data requiring processing
- **CDI Data**: Chemical Distribution Institute inspection records

### 2. Data Processing Pipeline

#### Vessel Filtering
The system applies intelligent filtering to ensure only relevant vessels are processed:

```python
# Vessel Type Filtering for SIRE Inspections
allowed_vessel_types = [
    "LNG CARRIER", "LPG CARRIER", "CHEM/PROD TANKER",
    "CRUDE OIL / PRODUCT CARRIER", "TANKER", "CHEMICAL CARRIER",
    "GAS CARRIER", "OIL TANKER"
]

# Filter active vessels of applicable types
filtered_vessels = vessels[
    (vessels['status'] == 'ACTIVE') &
    (vessels['vessel_type'].isin(allowed_vessel_types))
]
```

#### Date Validation and Sorting
Inspection dates are validated and sorted to identify the most recent inspections:

```python
# Convert and validate inspection dates
df['inspection_date'] = pd.to_datetime(df['inspection_date'], errors='coerce')

# Sort by most recent inspection first
df_sorted = df.sort_values('inspection_date', ascending=False)
```

### 3. Validity Period Calculations

#### SIRE Inspection Validity
SIRE inspections are valid for **180 days** from the inspection date:

```python
# Calculate SIRE validity period (180 days)
if inspection_date:
    valid_till_date = inspection_date + timedelta(days=180)
    next_due_days = (valid_till_date - today).days
    next_due_status = f"{next_due_days} days" if next_due_days >= 0 else "Overdue"
```

#### CDI Inspection Validity
CDI inspections are valid for **365 days** from the inspection date:

```python
# Calculate CDI validity period (365 days)
if inspection_date:
    valid_till_date = inspection_date + timedelta(days=365)
    next_due_days = (valid_till_date - today).days
    next_due_status = f"{next_due_days} days" if next_due_days >= 0 else "Overdue"
```

### 4. Statistical Analysis

#### Observation Counting by Chapter
The system analyzes inspection observations by chapter for trend identification:

```python
# Count observations by chapter
def calculate_chapter_statistics(observations):
    chapter_counts = {}
    total_observations = 0
    
    for observation in observations:
        question_number = observation.get('questionNumber', '')
        if question_number:
            # Extract chapter (part before decimal point)
            chapter = int(str(question_number).split('.')[0])
            chapter_counts[chapter] = chapter_counts.get(chapter, 0) + 1
            total_observations += 1
    
    return chapter_counts, total_observations
```

#### Fleet Performance Metrics
For fleet-level analysis, the system calculates:

```python
# Calculate fleet inspection metrics
total_inspections = len(inspection_data)
total_observations = sum(len(record['observations']) for record in inspection_data)
average_observations = total_observations / total_inspections if total_inspections > 0 else 0

# Identify performance trends
max_chapter = max(chapter_counts.items(), key=lambda x: x[1])[0]
min_chapter = min(chapter_counts.items(), key=lambda x: x[1])[0]
```

## Report Generation

### Individual Vessel Reports
Each vessel receives a comprehensive inspection report including:
- **Inspection History**: Chronological list of all inspections
- **Current Status**: Validity period and next due date
- **Observation Analysis**: Detailed breakdown of findings by chapter
- **Interactive Charts**: Visual representation of inspection trends

### Fleet Summary Reports
Fleet managers receive consolidated reports featuring:
- **Fleet Overview**: Summary statistics across all vessels
- **Compliance Status**: Vessels approaching inspection due dates
- **Performance Metrics**: Comparative analysis of observation trends
- **Data Tables**: Sortable and filterable inspection records

## Visual Analytics

### Interactive Charts
The system generates interactive charts using Highcharts with:
- **Main View**: Observations grouped by inspection chapter
- **Drill-down Capability**: Detailed view of specific question numbers within each chapter
- **Color Coding**: Visual differentiation of chapters and observation levels

### Data Tables
Comprehensive data tables provide:
- **Sortable Columns**: Easy organization of inspection data
- **Expandable Rows**: Detailed observation information on demand
- **Export Functionality**: Data export capabilities for further analysis

## Data Quality Assurance

### Validation Processes
- **Date Validation**: Ensures inspection dates are valid and properly formatted
- **Data Completeness**: Identifies and handles missing or incomplete records
- **Type Consistency**: Validates vessel types against approved inspection categories

### Error Handling
The system includes robust error handling for:
- **Missing Data**: Graceful handling of vessels without inspection records
- **Invalid Formats**: Automatic correction or flagging of data format issues
- **Processing Errors**: Comprehensive logging and error reporting

## Benefits for Maritime Stakeholders

### For Vessel Operators
- **Compliance Monitoring**: Real-time tracking of inspection validity periods
- **Performance Insights**: Understanding of inspection trends and areas for improvement
- **Operational Planning**: Advanced notice of upcoming inspection requirements

### For Fleet Managers
- **Portfolio Overview**: Comprehensive view of fleet inspection status
- **Risk Management**: Early identification of compliance risks
- **Resource Planning**: Optimized scheduling of inspection activities

### For Maritime Authorities
- **Industry Analytics**: Aggregated insights into inspection trends
- **Compliance Tracking**: Systematic monitoring of industry compliance levels
- **Performance Benchmarking**: Comparative analysis across vessel types and operators

## Technical Implementation

### Database Architecture
The system utilizes a hybrid architecture combining external APIs with internal database processing:
- **External APIs**: OCIMF and CDI website APIs for real-time inspection data retrieval
- **Data Ingestion**: API responses stored in multiple ERP databases (MongoDB collections)
- **Processing Layer**: Intermediate databases for data transformation and validation
- **Reporting Layer**: Optimized databases for fast report generation and component storage

### Scalability Features
- **Parallel Processing**: Concurrent processing of multiple vessels
- **Incremental Updates**: Efficient processing of only changed data
- **Caching Mechanisms**: Optimized performance for frequently accessed reports

### Security and Compliance
- **Data Encryption**: All data transmissions are encrypted
- **Access Controls**: Role-based access to sensitive inspection data
- **Audit Trails**: Comprehensive logging of all data access and modifications

## Conclusion

Our Maritime Inspection Data Processing System provides a comprehensive solution for managing and analyzing vessel inspection data. By combining SIRE and CDI inspection information with advanced analytics and visualization capabilities, the system enables maritime stakeholders to make informed decisions about vessel operations, compliance management, and risk mitigation.

The system's automated processing capabilities, combined with its flexible reporting features, ensure that users have access to timely, accurate, and actionable inspection data to support their maritime operations and regulatory compliance requirements.
