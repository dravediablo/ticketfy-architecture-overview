# ticketfy-architecture-overview

Offline-First Ticket Processing Architecture

Case Study – David Chacón

Executive Summary

This repository presents the architecture of an offline-first mobile application designed to process purchase receipts using OCR and AI services, synchronize structured financial data between local and cloud databases, and ensure reliable multi-device consistency.

The system combines:

A Flutter mobile application with a local SQLite database

Firebase Firestore as the cloud source of truth

A serverless backend deployed on Google Cloud Run

REST-based integrations with AI and OCR services

A bidirectional synchronization model with hybrid ID mapping

This case study demonstrates system design decisions, synchronization strategies, and API integration patterns relevant to scalable mobile platforms.

System Overview

The architecture follows a hybrid offline-first model:

Flutter App (SQLite)  ⇄  Firebase Firestore  ⇄  Cloud Run Backend
Core Principles

Local-first data access for performance and offline reliability

Cloud as authoritative consistency layer

Event-driven synchronization

Stateless backend processing

Clear separation between processing, storage, and presentation

Architecture Components
1. Mobile Application (Flutter + SQLite)

The mobile layer provides:

Full offline functionality

Local relational storage via SQLite

Real-time listeners to Firestore

Background synchronization logic

Conflict resolution handling

SQLite is used for:

Fast JOIN-based queries

Aggregations and reporting

Local caching of entities (categories, vendors, payment methods)

2. Cloud Database (Firebase Firestore)

Firestore acts as:

Global source of truth

Real-time synchronization hub

Multi-device consistency layer

User-isolated document storage

Data structure:

users/{userId}/
  tickets/{ticketId}
  vendors/{vendorId}
  categories/{categoryId}
  payment_methods/{paymentId}
3. Serverless Backend (Google Cloud Run)

The backend is responsible for:

Receiving receipt images via REST endpoints

Running OCR processing (Google Vision API)

Extracting structured data using AI services

Performing fuzzy entity matching

Writing normalized records to Firestore

The backend is stateless and horizontally scalable.

Offline-First Synchronization Strategy
Bidirectional Sync Model

Cloud → Local

Firestore listener detects changes

Ticket is retrieved

Hybrid ID mapping is resolved

Record is stored in SQLite

UI updates immediately

Local → Cloud

User edits ticket locally

SQLite updates

Firestore is updated asynchronously

Other devices receive changes

Hybrid ID Mapping Strategy

A hybrid ID structure is used to reconcile differences between:

Firestore document-based identifiers

SQLite relational numeric identifiers

Example:

SQLite:

id: 2
id_firebase: "firebase_vendor_123"

Ticket record:

vendor_id_local: 2
vendor_id_firebase: "firebase_vendor_123"

This enables:

Efficient relational JOINs locally

Consistent cloud references

Deterministic mapping across environments

Example SQL Queries

Example aggregation query:

SELECT category_id, SUM(total) AS total_spent
FROM tickets
GROUP BY category_id
ORDER BY total_spent DESC;

Example relational JOIN:

SELECT t.id, v.name, t.total
FROM tickets t
JOIN vendors v ON t.vendor_id_local = v.id
WHERE t.total > 500;

These queries support analytics, reporting, and budgeting features within the mobile application.

Example REST Integration (Node.js)
import fetch from 'node-fetch';

async function processReceipt(imageBase64) {
  const response = await fetch("https://api.example.com/process", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ image: imageBase64 })
  });

  if (!response.ok) {
    throw new Error("Processing failed");
  }

  return await response.json();
}

This demonstrates the API-driven integration pattern between the mobile client and backend processing service.

Conflict Resolution Strategy

Firestore server timestamps determine final write precedence

Local changes are queued while offline

Duplicate prevention logic runs before synchronization

Sync occurs incrementally based on last update timestamp

Performance Considerations

Indexed SQLite tables for fast local queries

Incremental synchronization (delta-based updates)

Image compression before upload

Stateless backend for horizontal scalability

Target performance benchmarks:

Image processing < 5 seconds

Sync propagation < 2 seconds

100% offline functional coverage

Security Model

Firebase Authentication

JWT validation for backend access

Per-user document isolation

Subscription-based usage validation

Key Technical Takeaways

This architecture demonstrates:

Offline-first mobile design

Cloud synchronization patterns

Hybrid relational-document mapping

Serverless backend integration

AI-driven structured data extraction

Multi-device consistency handling

About

This repository is a technical case study derived from the architecture of a production mobile system built and maintained by David Chacón.

It is intended to demonstrate applied system design, API integration, and synchronization strategies in modern mobile-cloud environments.
