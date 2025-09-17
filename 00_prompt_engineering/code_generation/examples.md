# Code Generation Examples: Real-World Templates and Prompts

*Practical Examples for GitHub Copilot and Cursor*

This document provides real-world examples and templates for effective AI code generation across different programming languages and scenarios.

---

## Table of Contents

1. [Python Examples](#python-examples)
2. [JavaScript/TypeScript Examples](#javascripttypescript-examples)
3. [React Examples](#react-examples)
4. [Node.js Examples](#nodejs-examples)
5. [Java Examples](#java-examples)
6. [Go Examples](#go-examples)
7. [Database Examples](#database-examples)
8. [Testing Examples](#testing-examples)
9. [DevOps Examples](#devops-examples)
10. [Advanced Patterns](#advanced-patterns)

---

## Python Examples

### 1. Data Processing Pipeline

**Prompt:**
```python
# Create a data processing pipeline that:
# 1. Reads CSV files from a directory
# 2. Validates data quality (missing values, data types)
# 3. Performs data transformations (normalization, encoding)
# 4. Generates summary statistics and visualizations
# 5. Exports cleaned data to multiple formats (CSV, JSON, Parquet)
# 6. Includes comprehensive logging and error handling
# 7. Supports parallel processing for large datasets

import pandas as pd
import numpy as np
import logging
from pathlib import Path
from concurrent.futures import ThreadPoolExecutor
import json
import pyarrow as pa
import pyarrow.parquet as pq

class DataProcessor:
    # AI will generate complete implementation
```

**Expected Output:**
```python
class DataProcessor:
    def __init__(self, input_dir: str, output_dir: str):
        self.input_dir = Path(input_dir)
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(exist_ok=True)
        self.setup_logging()
    
    def setup_logging(self):
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s',
            handlers=[
                logging.FileHandler('data_processing.log'),
                logging.StreamHandler()
            ]
        )
        self.logger = logging.getLogger(__name__)
    
    def process_files(self, max_workers: int = 4):
        """Process all CSV files in parallel"""
        csv_files = list(self.input_dir.glob('*.csv'))
        self.logger.info(f"Found {len(csv_files)} CSV files to process")
        
        with ThreadPoolExecutor(max_workers=max_workers) as executor:
            results = list(executor.map(self.process_single_file, csv_files))
        
        self.generate_summary_report(results)
        return results
    
    def process_single_file(self, file_path: Path):
        """Process a single CSV file"""
        try:
            self.logger.info(f"Processing {file_path.name}")
            
            # Read CSV with error handling
            df = pd.read_csv(file_path, encoding='utf-8')
            
            # Data quality validation
            quality_report = self.validate_data_quality(df)
            
            # Data transformations
            df_cleaned = self.transform_data(df)
            
            # Export to multiple formats
            self.export_data(df_cleaned, file_path.stem)
            
            return {
                'file': file_path.name,
                'status': 'success',
                'rows_processed': len(df_cleaned),
                'quality_report': quality_report
            }
            
        except Exception as e:
            self.logger.error(f"Error processing {file_path.name}: {str(e)}")
            return {
                'file': file_path.name,
                'status': 'error',
                'error': str(e)
            }
    
    def validate_data_quality(self, df: pd.DataFrame) -> dict:
        """Validate data quality and return report"""
        report = {
            'total_rows': len(df),
            'missing_values': df.isnull().sum().to_dict(),
            'duplicate_rows': df.duplicated().sum(),
            'data_types': df.dtypes.to_dict(),
            'memory_usage': df.memory_usage(deep=True).sum()
        }
        
        # Check for outliers in numeric columns
        numeric_cols = df.select_dtypes(include=[np.number]).columns
        outliers = {}
        for col in numeric_cols:
            Q1 = df[col].quantile(0.25)
            Q3 = df[col].quantile(0.75)
            IQR = Q3 - Q1
            outliers[col] = len(df[(df[col] < Q1 - 1.5*IQR) | (df[col] > Q3 + 1.5*IQR)])
        
        report['outliers'] = outliers
        return report
    
    def transform_data(self, df: pd.DataFrame) -> pd.DataFrame:
        """Apply data transformations"""
        df_cleaned = df.copy()
        
        # Handle missing values
        numeric_cols = df_cleaned.select_dtypes(include=[np.number]).columns
        df_cleaned[numeric_cols] = df_cleaned[numeric_cols].fillna(df_cleaned[numeric_cols].mean())
        
        categorical_cols = df_cleaned.select_dtypes(include=['object']).columns
        df_cleaned[categorical_cols] = df_cleaned[categorical_cols].fillna('Unknown')
        
        # Remove duplicates
        df_cleaned = df_cleaned.drop_duplicates()
        
        # Normalize numeric columns
        for col in numeric_cols:
            if df_cleaned[col].std() != 0:
                df_cleaned[col] = (df_cleaned[col] - df_cleaned[col].mean()) / df_cleaned[col].std()
        
        return df_cleaned
    
    def export_data(self, df: pd.DataFrame, filename: str):
        """Export data to multiple formats"""
        # CSV
        df.to_csv(self.output_dir / f"{filename}_cleaned.csv", index=False)
        
        # JSON
        df.to_json(self.output_dir / f"{filename}_cleaned.json", orient='records', indent=2)
        
        # Parquet
        table = pa.Table.from_pandas(df)
        pq.write_table(table, self.output_dir / f"{filename}_cleaned.parquet")
        
        self.logger.info(f"Exported {filename} to multiple formats")
    
    def generate_summary_report(self, results: list):
        """Generate summary report of all processed files"""
        successful = [r for r in results if r['status'] == 'success']
        failed = [r for r in results if r['status'] == 'error']
        
        summary = {
            'total_files': len(results),
            'successful': len(successful),
            'failed': len(failed),
            'total_rows_processed': sum(r.get('rows_processed', 0) for r in successful),
            'failed_files': [r['file'] for r in failed]
        }
        
        with open(self.output_dir / 'processing_summary.json', 'w') as f:
            json.dump(summary, f, indent=2)
        
        self.logger.info(f"Processing complete: {summary}")
```

### 2. Machine Learning Pipeline

**Prompt:**
```python
# Create a complete machine learning pipeline for customer churn prediction:
# 1. Data loading and preprocessing
# 2. Feature engineering and selection
# 3. Model training with multiple algorithms (Random Forest, XGBoost, Neural Network)
# 4. Hyperparameter tuning with cross-validation
# 5. Model evaluation with multiple metrics
# 6. Model persistence and loading
# 7. Prediction pipeline for new data
# 8. Comprehensive logging and monitoring

import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split, cross_val_score, GridSearchCV
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, confusion_matrix, roc_auc_score
from sklearn.preprocessing import StandardScaler, LabelEncoder
import xgboost as xgb
import joblib
import logging
from typing import Dict, List, Tuple, Any

class ChurnPredictionPipeline:
    # AI will generate complete implementation
```

### 3. REST API with FastAPI

**Prompt:**
```python
# Create a FastAPI REST API for a user management system with:
# 1. User CRUD operations (Create, Read, Update, Delete)
# 2. JWT authentication and authorization
# 3. Password hashing with bcrypt
# 4. Input validation with Pydantic models
# 5. Database integration with SQLAlchemy
# 6. Error handling and logging
# 7. API documentation with Swagger
# 8. Rate limiting and security middleware
# 9. Unit tests with pytest
# 10. Docker containerization

from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from fastapi.middleware.cors import CORSMiddleware
from sqlalchemy import create_engine, Column, Integer, String, DateTime, Boolean
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from pydantic import BaseModel, EmailStr
from passlib.context import CryptContext
from jose import JWTError, jwt
from datetime import datetime, timedelta
import logging
from typing import Optional, List

# AI will generate complete implementation
```

---

## JavaScript/TypeScript Examples

### 1. React Component with Hooks

**Prompt:**
```typescript
// Create a React TypeScript component for a data table with:
// - Sortable columns with custom sorting logic
// - Filtering by multiple criteria (text, date range, dropdown)
// - Pagination with configurable page sizes
// - Export functionality (CSV, Excel, PDF)
// - Responsive design with Tailwind CSS
// - Loading states and error handling
// - Accessibility features (ARIA labels, keyboard navigation)
// - Custom hooks for data management
// - TypeScript interfaces for type safety

import React, { useState, useEffect, useMemo, useCallback } from 'react';
import { ChevronUpIcon, ChevronDownIcon, FunnelIcon } from '@heroicons/react/24/outline';

interface DataTableProps<T> {
  data: T[];
  columns: ColumnDef<T>[];
  onSort?: (column: string, direction: 'asc' | 'desc') => void;
  onFilter?: (filters: FilterState) => void;
  loading?: boolean;
  error?: string;
}

// AI will generate complete implementation
```

**Expected Output:**
```typescript
interface ColumnDef<T> {
  key: keyof T;
  label: string;
  sortable?: boolean;
  filterable?: boolean;
  render?: (value: any, row: T) => React.ReactNode;
  width?: string;
}

interface FilterState {
  [key: string]: string | number | Date | null;
}

interface SortState {
  column: string | null;
  direction: 'asc' | 'desc' | null;
}

export function DataTable<T extends Record<string, any>>({
  data,
  columns,
  onSort,
  onFilter,
  loading = false,
  error
}: DataTableProps<T>) {
  const [sortState, setSortState] = useState<SortState>({ column: null, direction: null });
  const [filters, setFilters] = useState<FilterState>({});
  const [currentPage, setCurrentPage] = useState(1);
  const [pageSize, setPageSize] = useState(10);
  const [selectedRows, setSelectedRows] = useState<Set<string>>(new Set());

  // Filtered and sorted data
  const processedData = useMemo(() => {
    let filtered = data.filter(row => {
      return Object.entries(filters).every(([key, value]) => {
        if (!value) return true;
        const cellValue = row[key];
        if (typeof cellValue === 'string') {
          return cellValue.toLowerCase().includes(String(value).toLowerCase());
        }
        return cellValue === value;
      });
    });

    if (sortState.column) {
      filtered.sort((a, b) => {
        const aVal = a[sortState.column!];
        const bVal = b[sortState.column!];
        
        if (aVal < bVal) return sortState.direction === 'asc' ? -1 : 1;
        if (aVal > bVal) return sortState.direction === 'asc' ? 1 : -1;
        return 0;
      });
    }

    return filtered;
  }, [data, filters, sortState]);

  // Pagination
  const totalPages = Math.ceil(processedData.length / pageSize);
  const paginatedData = processedData.slice(
    (currentPage - 1) * pageSize,
    currentPage * pageSize
  );

  const handleSort = useCallback((column: string) => {
    const newDirection = sortState.column === column && sortState.direction === 'asc' 
      ? 'desc' 
      : 'asc';
    
    setSortState({ column, direction: newDirection });
    onSort?.(column, newDirection);
  }, [sortState, onSort]);

  const handleFilter = useCallback((key: string, value: string | number | Date | null) => {
    const newFilters = { ...filters, [key]: value };
    setFilters(newFilters);
    onFilter?.(newFilters);
    setCurrentPage(1); // Reset to first page when filtering
  }, [filters, onFilter]);

  const exportToCSV = useCallback(() => {
    const headers = columns.map(col => col.label).join(',');
    const rows = processedData.map(row => 
      columns.map(col => `"${row[col.key] || ''}"`).join(',')
    );
    const csvContent = [headers, ...rows].join('\n');
    
    const blob = new Blob([csvContent], { type: 'text/csv' });
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.download = 'data.csv';
    link.click();
    URL.revokeObjectURL(url);
  }, [processedData, columns]);

  if (loading) {
    return (
      <div className="flex justify-center items-center h-64">
        <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
      </div>
    );
  }

  if (error) {
    return (
      <div className="bg-red-50 border border-red-200 rounded-md p-4">
        <p className="text-red-800">Error: {error}</p>
      </div>
    );
  }

  return (
    <div className="bg-white shadow-lg rounded-lg overflow-hidden">
      {/* Header with filters and export */}
      <div className="p-4 border-b border-gray-200">
        <div className="flex justify-between items-center mb-4">
          <h3 className="text-lg font-semibold text-gray-900">Data Table</h3>
          <div className="flex space-x-2">
            <button
              onClick={exportToCSV}
              className="px-4 py-2 bg-green-600 text-white rounded-md hover:bg-green-700 transition-colors"
            >
              Export CSV
            </button>
          </div>
        </div>
        
        {/* Filters */}
        <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
          {columns.filter(col => col.filterable).map(column => (
            <div key={String(column.key)}>
              <label className="block text-sm font-medium text-gray-700 mb-1">
                {column.label}
              </label>
              <input
                type="text"
                placeholder={`Filter by ${column.label}`}
                value={filters[String(column.key)] || ''}
                onChange={(e) => handleFilter(String(column.key), e.target.value)}
                className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </div>
          ))}
        </div>
      </div>

      {/* Table */}
      <div className="overflow-x-auto">
        <table className="min-w-full divide-y divide-gray-200">
          <thead className="bg-gray-50">
            <tr>
              {columns.map(column => (
                <th
                  key={String(column.key)}
                  className={`px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider ${
                    column.sortable ? 'cursor-pointer hover:bg-gray-100' : ''
                  }`}
                  onClick={() => column.sortable && handleSort(String(column.key))}
                  style={{ width: column.width }}
                >
                  <div className="flex items-center space-x-1">
                    <span>{column.label}</span>
                    {column.sortable && (
                      <div className="flex flex-col">
                        <ChevronUpIcon 
                          className={`h-3 w-3 ${
                            sortState.column === String(column.key) && sortState.direction === 'asc'
                              ? 'text-blue-600' : 'text-gray-400'
                          }`} 
                        />
                        <ChevronDownIcon 
                          className={`h-3 w-3 -mt-1 ${
                            sortState.column === String(column.key) && sortState.direction === 'desc'
                              ? 'text-blue-600' : 'text-gray-400'
                          }`} 
                        />
                      </div>
                    )}
                  </div>
                </th>
              ))}
            </tr>
          </thead>
          <tbody className="bg-white divide-y divide-gray-200">
            {paginatedData.map((row, index) => (
              <tr key={index} className="hover:bg-gray-50">
                {columns.map(column => (
                  <td key={String(column.key)} className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                    {column.render 
                      ? column.render(row[column.key], row)
                      : row[column.key]
                    }
                  </td>
                ))}
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      {/* Pagination */}
      <div className="px-6 py-3 border-t border-gray-200">
        <div className="flex items-center justify-between">
          <div className="flex items-center space-x-2">
            <span className="text-sm text-gray-700">Rows per page:</span>
            <select
              value={pageSize}
              onChange={(e) => {
                setPageSize(Number(e.target.value));
                setCurrentPage(1);
              }}
              className="px-3 py-1 border border-gray-300 rounded-md text-sm"
            >
              <option value={10}>10</option>
              <option value={25}>25</option>
              <option value={50}>50</option>
              <option value={100}>100</option>
            </select>
          </div>
          
          <div className="flex items-center space-x-2">
            <span className="text-sm text-gray-700">
              Showing {((currentPage - 1) * pageSize) + 1} to {Math.min(currentPage * pageSize, processedData.length)} of {processedData.length} results
            </span>
            
            <div className="flex space-x-1">
              <button
                onClick={() => setCurrentPage(prev => Math.max(1, prev - 1))}
                disabled={currentPage === 1}
                className="px-3 py-1 border border-gray-300 rounded-md text-sm disabled:opacity-50 disabled:cursor-not-allowed"
              >
                Previous
              </button>
              
              {Array.from({ length: Math.min(5, totalPages) }, (_, i) => {
                const page = i + 1;
                return (
                  <button
                    key={page}
                    onClick={() => setCurrentPage(page)}
                    className={`px-3 py-1 border rounded-md text-sm ${
                      currentPage === page
                        ? 'bg-blue-600 text-white border-blue-600'
                        : 'border-gray-300 text-gray-700 hover:bg-gray-50'
                    }`}
                  >
                    {page}
                  </button>
                );
              })}
              
              <button
                onClick={() => setCurrentPage(prev => Math.min(totalPages, prev + 1))}
                disabled={currentPage === totalPages}
                className="px-3 py-1 border border-gray-300 rounded-md text-sm disabled:opacity-50 disabled:cursor-not-allowed"
              >
                Next
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}
```

### 2. Node.js API with Express

**Prompt:**
```javascript
// Create a Node.js Express API for a task management system with:
// 1. RESTful endpoints for CRUD operations
// 2. JWT authentication middleware
// 3. Input validation with Joi
// 4. Database integration with Prisma ORM
// 5. Error handling and logging
// 6. Rate limiting and security headers
// 7. API documentation with Swagger
// 8. Unit tests with Jest
// 9. Docker containerization
// 10. Environment configuration

const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const { PrismaClient } = require('@prisma/client');
const Joi = require('joi');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');
const winston = require('winston');

// AI will generate complete implementation
```

---

## React Examples

### 1. Custom Hook for API Management

**Prompt:**
```typescript
// Create a custom React hook for API state management with:
// 1. Loading, error, and success states
// 2. Automatic retry logic with exponential backoff
// 3. Request cancellation with AbortController
// 4. Caching with configurable TTL
// 5. Optimistic updates
// 6. TypeScript support with generics
// 7. Error boundary integration
// 8. Request deduplication
// 9. Background refresh
// 10. Comprehensive logging

import { useState, useEffect, useCallback, useRef } from 'react';

interface UseApiOptions<T> {
  retryAttempts?: number;
  retryDelay?: number;
  cacheTime?: number;
  staleTime?: number;
  enableBackgroundRefresh?: boolean;
  onSuccess?: (data: T) => void;
  onError?: (error: Error) => void;
}

interface ApiState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  isStale: boolean;
  lastFetched: Date | null;
}

// AI will generate complete implementation
```

### 2. Form Management with Validation

**Prompt:**
```typescript
// Create a comprehensive form management system with:
// 1. Dynamic form fields with conditional logic
// 2. Real-time validation with custom rules
// 3. Field dependencies and cascading updates
// 4. File upload with progress tracking
// 5. Form state persistence (localStorage/sessionStorage)
// 6. Accessibility features (ARIA labels, keyboard navigation)
// 7. Multi-step form wizard
// 8. Form submission with optimistic updates
// 9. Error handling and user feedback
// 10. TypeScript support with strict typing

import React, { useState, useEffect, useCallback } from 'react';
import { useForm, Controller, useFieldArray } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';

interface FormField {
  name: string;
  type: 'text' | 'email' | 'password' | 'select' | 'checkbox' | 'file' | 'textarea';
  label: string;
  required?: boolean;
  validation?: yup.Schema;
  options?: { value: string; label: string }[];
  dependsOn?: string;
  conditional?: (values: any) => boolean;
}

interface FormConfig {
  fields: FormField[];
  steps?: string[];
  onSubmit: (data: any) => Promise<void>;
  onSave?: (data: any) => void;
  persistKey?: string;
}

// AI will generate complete implementation
```

---

## Node.js Examples

### 1. Microservice Architecture

**Prompt:**
```javascript
// Create a microservice for user management with:
// 1. Express.js REST API
// 2. PostgreSQL database with Prisma ORM
// 3. Redis for caching and session management
// 4. JWT authentication and authorization
// 5. Input validation with Joi
// 6. Error handling and logging with Winston
// 7. Rate limiting and security middleware
// 8. Health checks and monitoring
// 9. Docker containerization
// 10. Unit and integration tests

const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const { PrismaClient } = require('@prisma/client');
const redis = require('redis');
const Joi = require('joi');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');
const winston = require('winston');

// AI will generate complete implementation
```

### 2. WebSocket Real-time Chat

**Prompt:**
```javascript
// Create a real-time chat application with:
// 1. WebSocket server with Socket.io
// 2. Room-based messaging system
// 3. User authentication and authorization
// 4. Message persistence with MongoDB
// 5. File sharing capabilities
// 6. Typing indicators and online status
// 7. Message encryption for privacy
// 8. Rate limiting and spam protection
// 9. Message moderation and reporting
// 10. Mobile-responsive frontend

const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const mongoose = require('mongoose');
const multer = require('multer');
const crypto = require('crypto');

// AI will generate complete implementation
```

---

## Java Examples

### 1. Spring Boot REST API

**Prompt:**
```java
// Create a Spring Boot REST API for an e-commerce system with:
// 1. RESTful endpoints for products, orders, and users
// 2. JPA entities with proper relationships
// 3. Service layer with business logic
// 4. Repository layer with custom queries
// 5. Input validation with Bean Validation
// 6. Exception handling with @ControllerAdvice
// 7. Security with Spring Security and JWT
// 8. API documentation with Swagger/OpenAPI
// 9. Unit tests with JUnit and Mockito
// 10. Integration tests with TestContainers

@SpringBootApplication
@EnableJpaRepositories
@EnableWebSecurity
@OpenAPIDefinition(info = @Info(title = "E-commerce API", version = "1.0"))
public class ECommerceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ECommerceApplication.class, args);
    }
}

// AI will generate complete implementation
```

### 2. Microservices with Spring Cloud

**Prompt:**
```java
// Create a microservices architecture with Spring Cloud:
// 1. Service discovery with Eureka
// 2. API Gateway with Spring Cloud Gateway
// 3. Configuration management with Spring Cloud Config
// 4. Circuit breaker with Resilience4j
// 5. Distributed tracing with Sleuth and Zipkin
// 6. Message-driven communication with RabbitMQ
// 7. Security with OAuth2 and JWT
// 8. Monitoring with Micrometer and Prometheus
// 9. Docker containerization
// 10. Kubernetes deployment manifests

@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}

// AI will generate complete implementation
```

---

## Go Examples

### 1. REST API with Gin

**Prompt:**
```go
// Create a Go REST API with Gin framework for a blog system:
// 1. RESTful endpoints for posts, comments, and users
// 2. Database integration with GORM
// 3. JWT authentication middleware
// 4. Input validation and sanitization
// 5. Error handling and logging
// 6. Rate limiting and security headers
// 7. API documentation with Swagger
// 8. Unit tests with testify
// 9. Docker containerization
// 10. Graceful shutdown and health checks

package main

import (
    "github.com/gin-gonic/gin"
    "github.com/golang-jwt/jwt/v5"
    "gorm.io/gorm"
    "log"
    "net/http"
)

// AI will generate complete implementation
```

### 2. Concurrent Worker Pool

**Prompt:**
```go
// Create a concurrent worker pool system in Go:
// 1. Configurable number of workers
// 2. Job queue with channels
// 3. Graceful shutdown handling
// 4. Worker health monitoring
// 5. Job result collection
// 6. Error handling and retry logic
// 7. Metrics and monitoring
// 8. Context cancellation support
// 9. Priority-based job scheduling
// 10. Comprehensive logging

package main

import (
    "context"
    "fmt"
    "log"
    "sync"
    "time"
)

// AI will generate complete implementation
```

---

## Database Examples

### 1. SQL Schema Design

**Prompt:**
```sql
-- Create a comprehensive database schema for an e-commerce platform:
-- 1. Users table with authentication fields
-- 2. Products table with categories and variants
-- 3. Orders table with order items
-- 4. Payments table with transaction history
-- 5. Reviews and ratings system
-- 6. Inventory management
-- 7. Proper indexing for performance
-- 8. Foreign key constraints
-- 9. Triggers for data consistency
-- 10. Views for common queries

-- AI will generate complete schema
```

### 2. NoSQL Document Design

**Prompt:**
```javascript
// Create MongoDB collections for a social media platform:
// 1. Users collection with embedded profiles
// 2. Posts collection with comments and likes
// 3. Messages collection for direct messaging
// 4. Notifications collection for real-time updates
// 5. Proper indexing strategies
// 6. Data validation schemas
// 7. Aggregation pipelines for analytics
// 8. Text search capabilities
// 9. Geospatial queries for location features
// 10. Data archiving and cleanup strategies

// AI will generate complete MongoDB schema
```

---

## Testing Examples

### 1. Unit Testing with Jest

**Prompt:**
```javascript
// Create comprehensive unit tests for a user service with:
// 1. Test setup and teardown
// 2. Mocking external dependencies
// 3. Testing success and error scenarios
// 4. Edge cases and boundary conditions
// 5. Async function testing
// 6. Test coverage reporting
// 7. Parameterized tests
// 8. Snapshot testing for components
// 9. Integration with CI/CD pipeline
// 10. Performance testing

import { UserService } from '../services/UserService';
import { UserRepository } from '../repositories/UserRepository';
import { EmailService } from '../services/EmailService';

// AI will generate complete test suite
```

### 2. Integration Testing

**Prompt:**
```python
# Create integration tests for a FastAPI application with:
# 1. Test database setup and cleanup
# 2. API endpoint testing
# 3. Authentication flow testing
# 4. Database transaction testing
# 5. External service mocking
# 6. Test data factories
# 7. Parallel test execution
# 8. Test reporting and coverage
# 9. Performance benchmarking
# 10. End-to-end workflow testing

import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.main import app, get_db
from app.database import Base

# AI will generate complete test suite
```

---

## DevOps Examples

### 1. Docker Configuration

**Prompt:**
```dockerfile
# Create a multi-stage Dockerfile for a Node.js application with:
# 1. Multi-stage build for optimization
# 2. Security best practices
# 3. Health checks
# 4. Non-root user
# 5. Proper layer caching
# 6. Environment variable handling
# 7. Build optimization
# 8. Security scanning
# 9. Multi-architecture support
# 10. Production-ready configuration

# AI will generate complete Dockerfile
```

### 2. Kubernetes Deployment

**Prompt:**
```yaml
# Create Kubernetes manifests for a microservices application:
# 1. Deployment configurations
# 2. Service definitions
# 3. Ingress configuration
# 4. ConfigMaps and Secrets
# 5. Persistent volumes
# 6. Resource limits and requests
# 7. Health checks and probes
# 8. Horizontal Pod Autoscaling
# 9. Network policies
# 10. Monitoring and logging

# AI will generate complete Kubernetes manifests
```

---

## Advanced Patterns

### 1. Design Pattern Implementation

**Prompt:**
```typescript
// Implement the Observer pattern for a real-time notification system:
// 1. Subject interface for observable objects
// 2. Observer interface for subscribers
// 3. Concrete subject implementation
// 4. Concrete observer implementations
// 5. Event handling and propagation
// 6. Memory management and cleanup
// 7. TypeScript generics for type safety
// 8. Error handling and recovery
// 9. Performance optimization
// 10. Unit tests and documentation

interface Subject<T> {
  attach(observer: Observer<T>): void;
  detach(observer: Observer<T>): void;
  notify(data: T): void;
}

interface Observer<T> {
  update(data: T): void;
}

// AI will generate complete implementation
```

### 2. Functional Programming Patterns

**Prompt:**
```javascript
// Create functional programming utilities with:
// 1. Pure functions for data transformation
// 2. Higher-order functions for composition
// 3. Immutable data structures
// 4. Function currying and partial application
// 5. Monadic error handling
// 6. Lazy evaluation and memoization
// 7. Function pipelines and chaining
// 8. TypeScript support with generics
// 9. Performance optimization
// 10. Comprehensive testing

// AI will generate complete functional programming library
```

---

## Conclusion

These examples demonstrate the power of effective prompting for AI code generation. By providing clear, detailed instructions and context, you can generate high-quality, production-ready code across different programming languages and frameworks.

Remember to:
- **Be specific** about requirements and constraints
- **Provide context** about frameworks, libraries, and architecture
- **Include examples** of expected behavior and output
- **Specify testing** requirements and quality standards
- **Iterate and refine** prompts based on results

The key to successful AI code generation is understanding that these tools are most effective when given comprehensive, well-structured instructions that clearly communicate your intent and requirements.
