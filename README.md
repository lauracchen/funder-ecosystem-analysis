# Funder Network Ecosystem Analysis

A flexible starting point for analyzing and understanding the philanthropic ecosystem around the Robert Sterling Clark Foundation. This project was developed to help the Robert Sterling Clark Foundation (RSCF) better understand the funder network that supports similar grantee-partners, identify trends in grantmaking over time, and surface insights about the broader funding landscape. We also wanted to aggregate relevant information for grantee partners as they map funding sources.

## Overview

This analysis tool helps foundations and researchers:

- **Identify overlap** in the funder ecosystem: which foundations fund similar organizations to yours
- **Enrich funder data** using AI-powered web research to gather programmatic and geographic focus areas
- **Analyze funding trends** over time with inflation-adjusted historical data
- **Visualize the landscape** through treemaps, time series, and distribution charts
- **Surface partnership opportunities** by understanding which funders have similar missions and accept unsolicited proposals

The tool is designed to be **adaptable** - while it was built for RSCF's needs, it can be customized to explore different research questions about any funder network.

## Data Sources

This analysis integrates data from multiple sources:

1. **RSCF Grantee-Partner List**: The foundation's list of organizations it supports, used as the basis for identifying overlapping funders
2. **Impala Database**: Provides current data on funders who also support RSCF's grantee-partners (note: Impala is a proprietary database and not publicly accessible)
3. **ProPublica Nonprofit Explorer API**: Supplies historical IRS Form 990 data, enabling trend analysis of grantmaking over time

## Key Features

### AI-Powered Metadata Enrichment

One of the most powerful features of this tool is its use of **large language models (LLMs) to automatically research and enrich funder information**. The tool uses OpenAI's models with web search capabilities to automatically gather:

- **Geographic focus areas**: Where do funders concentrate their grantmaking?
- **Programmatic priorities**: Do they focus on leadership development, civic engagement, or other specific areas?
- **Application processes**: Do they accept unsolicited letters of interest?

**This approach is highly flexible** - you can modify the enrichment function to research any questions relevant to your work. For example, you could ask the LLM to identify:
- Whether funders support multi-year grants
- Their average grant size ranges
- Specific populations or communities they serve
- Their stance on general operating support
- Board diversity or staff size

The LLM automatically searches the web, prioritizes authoritative sources (organization websites, GuideStar), and returns structured data with citations - all without manual research.

### Funder Overlap Analysis

The tool identifies which foundations fund your grantee-partners and visualizes the degree of overlap through treemaps, helping you understand who the major co-funders are in your ecosystem.

### Historical Trend Analysis

By pulling IRS Form 990 data over multiple years and adjusting for inflation, the tool tracks how grantmaking budgets have changed over time - both for individual funders and across the entire network.

### Flexible Visualizations

The notebook generates multiple types of visualizations:
- Treemaps showing funder overlap
- Time series of funding trends
- Distribution charts of grant sizes and grantmaking budgets
- Comparative analyses across funding tiers

## How It Works

The general workflow is:

1. **Start with a list of funders** you want to analyze (from your database, a spreadsheet, or another source)
2. **Match funder names** to their official IRS names and retrieve their Employer Identification Numbers (EINs)
3. **Enrich the data** using LLM-powered web research to gather additional metadata
4. **Retrieve Form 990 data** from ProPublica's API to get historical grantmaking information
5. **Adjust for inflation** to enable fair comparisons across years
6. **Analyze and visualize** the data to surface insights

## Adapting This Tool for Your Needs

This tool is designed to be adapted for your own funder network research. The **main requirement** is:

- **A dataframe (or CSV file) containing funder names** - this is the starting point for all analysis

Optional enhancements include:
- Pre-existing metadata about funders (state, focus areas, etc.)
- Your own list of grantee-partners or organizations of interest
- Custom research questions for the LLM enrichment process

### Getting Started with Your Own Data

To adapt this tool:

1. **Prepare your funder list**: Create a CSV or load a dataframe with at least one column containing funder names
2. **Customize the enrichment questions**: Modify the `research_funder()` function to ask the research questions most relevant to your work
3. **Adjust the matching logic**: Depending on your data quality, you may need to tune the fuzzy matching parameters when aligning your funder names with IRS records
4. **Select your analysis period**: Choose which years of Form 990 data are most relevant for your analysis

The notebook includes comments and explanations throughout to guide customization.

## Setup Instructions

### Prerequisites

- Python 3.8 or higher
- An OpenAI API key (for the LLM-powered enrichment features)

### Installation

1. Clone this repository
2. Install required packages:
   ```bash
   pip install -r requirements.txt
   ```

3. Create a `.env` file based on `.env.example`:
   ```bash
   cp .env.example .env
   ```

4. Add your OpenAI API key to the `.env` file:
   ```
   OPENAI_API_KEY=your_actual_api_key_here
   ```

5. Prepare your data:
   - Create a `data/` directory in the project root
   - Place your funder list CSV in the `data/` directory
   - Update the file path references in the notebook

### Running the Analysis

Open `irs_data.ipynb` in Jupyter Notebook or JupyterLab and run the cells sequentially. The notebook is organized into logical sections that can be run independently once initial data loading is complete.

## The IRS XML Challenge: Why We Use the ProPublica API

### Initial Approach

We initially attempted to work directly with IRS XML files, which are publicly available through the IRS's bulk data downloads. This approach seemed attractive because:

- Direct access to the most current data, without waiting for others to process the data
- Complete field coverage with all Form 990 data points available
- No dependency on external paid services

### The Complexity Problem

However, we encountered significant challenges that made direct XML parsing impractical for this project:

**1. Schema Variations Across Years**

The IRS has modified the Form 990 XML schema multiple times over the years. Field names, structures, and nesting patterns changed between tax years, meaning that code written to parse 2023 filings might not work for 2015 filings without substantial modification.

**2. Inconsistent Field Naming**

Even within the same general time period, we found inconsistencies:
- The same concept might be represented with different XML element names
- Fields might appear in different sections of the form structure
- Some organizations' filings used different schema versions even for the same tax year

**3. Namespace and Structure Challenges**

The XML files use complex namespace definitions, and the location of key data points (like contributions paid) varied:
- Some filings nested the data under `AnalysisOfRevenueAndExpenses`
- Others used different parent elements
- Required extensive XML tree traversal with multiple fallback strategies

**4. Data Volume**

The IRS bulk files are enormous (multiple gigabytes per year), requiring significant download, storage, and processing time - especially when only a subset of organizations (83 funders in our case) were needed.

### Our Solution

Given these challenges, we determined that **direct IRS XML parsing was out of scope** for this project. Instead, we use:

- **ProPublica Nonprofit Explorer API** for historical Form 990 data: ProPublica has done the heavy lifting of parsing, standardizing, and structuring the IRS data across years
- **Impala database** for the most recent data: When available, proprietary databases often have more current information than publicly available sources

This approach provides:
- Consistent field names across years
- Clean, JSON-formatted data that's easy to work with
- Reasonable API rate limits for research purposes
- No need to manage large file downloads

**Note**: If you need to work with IRS XML files directly for your research, consider using dedicated parsing libraries or tools specifically designed for this purpose, as the schema complexity requires specialized handling.

## Outputs and Visualizations

The analysis produces several types of outputs:

1. **Enriched funder dataset** (CSV): Your original funder list enhanced with geographic focus, programmatic priorities, application information, and citations
2. **Treemap visualizations**: Show the relative overlap of funders within your network
3. **Time series charts**: Display inflation-adjusted funding trends over time
4. **Distribution analyses**: Illustrate the range of grantmaking budgets and median grant sizes
5. **Summary statistics**: Aggregate views of funding available across the network

All visualizations are generated inline in the Jupyter notebook and can be exported for presentations or reports.

## Questions or Issues

This tool was developed as an internal analysis project and is shared as a methodology demonstration. While we cannot provide ongoing support, we welcome questions about the approach and are happy to hear about adaptations you've made for your own work.

## Acknowledgments

This project uses:
- [ProPublica Nonprofit Explorer API](https://projects.propublica.org/nonprofits/api) for IRS Form 990 data
- OpenAI's GPT models for web research and data enrichment
- Various Python data science libraries (pandas, matplotlib, etc.)

Data on co-funders was sourced from the Impala database where available.