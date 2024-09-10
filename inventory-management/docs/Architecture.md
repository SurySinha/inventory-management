## Note: Please install Mermaid Extension for looking at the Diagrams. 

# Order Management System Analysis and Improved Architecture

## Current State Analysis

The current system architecture consists of the following components:

- Warehouse Management System (WMS)
- Online Storefront
- eBay Marketplace
- Amazon Marketplace
- Middleware System
- SFTP Server

### Key Characteristics:

1. Single inventory pool in WMS
2. Inventory updates every 15 minutes
3. Orders exported in batches every 15 minutes
4. Orders aggregated and posted to WMS every 15 minutes

### Current State Diagram

```mermaid
graph TD
    A[WMS] -->|Inventory deltas every 15 min| B[Middleware]
    B --> C[Online Storefront]
    B --> D[eBay Marketplace]
    B --> E[Amazon Marketplace]
    C -->|Orders every 15 min| F[SFTP Server]
    D -->|Orders every 15 min| F
    E -->|Orders every 15 min| F
    F -->|Aggregated orders every 15 min| B
    B -->|Orders| A
```

### Causes of Overselling

1. Delayed inventory updates
2. Batch processing of orders
3. Lack of real-time inventory tracking
4. No inventory reservation system

#### Quick Wins:
1. Increase update frequency: Reduce the interval for inventory updates and order processing from 15 minutes to 5 minutes or less.
2. Implement safety stock: Keep a buffer stock for each product to account for potential discrepancies.
3. Prioritize high-demand items: Implement more frequent updates for fast-moving products.


## Proposed DOM Architecture with Kafka

To address the overselling issues and improve overall system performance, we propose a new architecture based on a Distributed Order Management (DOM) system using Kafka as the central message bus.

### Enhanced DOM Architecture Diagram

```mermaid
graph TD
   subgraph External Systems
      B[Online Storefront]
      C[eBay Marketplace]
      D[Amazon Marketplace]
   end

   subgraph API Layer
      M[API Gateway]
      R[REST APIs]
   end

   subgraph Internal Architecture
      subgraph Kafka Ecosystem
         E[Kafka Cluster]
         F[Kafka Connect]
         G[Schema Registry]
      end

      subgraph DOM Core Services
         H[Inventory Service]
         I[Order Service]
         J[Order Orchestration]
         K[Analytics Service]
      end

      A[WMS]
      L[Notification Service]
      N[Admin Dashboard]
   end

   O[Monitoring & Alerting]

   B --> M
   C --> M
   D --> M
   M --> R
   R <--> H & I & J & K
   A <--> E
   H & I & J & K <--> E
   L <--> E
   E <--> F
   E <--> G
   N <--> E

   O -.-> B & C & D
   O -.-> M & R
   O -.-> E & F & G
   O -.-> H & I & J & K & A & L & N

   classDef external fill:#e1f5fe,stroke:#01579b
   classDef api fill:#fff9c4,stroke:#fbc02d
   classDef kafka fill:#f1f8e9,stroke:#33691e
   classDef core fill:#fff3e0,stroke:#e65100
   classDef support fill:#f3e5f5,stroke:#4a148c
   classDef monitoring fill:#fbe9e7,stroke:#bf360c
   classDef wms fill:#dcedc8,stroke:#33691e

   class B,C,D external
   class M,R api
   class E,F,G kafka
   class H,I,J,K core
   class L,N support
   class O monitoring
   class A wms
```

### Key Components:

1. **REST Services:**
    - Design RESTful APIs that encapsulate all necessary operations for external systems.
2. **Kafka Ecosystem:**
    - Kafka Cluster: Central message bus for all event streaming
    - Kafka Connect: Facilitates integration with external systems

2. **DOM Core Services:**
    - Inventory Service: Manages real-time inventory state
    - Order Service: Handles order lifecycle
    - Order Orchestration: Implements routing and fulfillment logic
    - Analytics Service: Processes events for insights and reporting

3. **Support Services:**
    - Notification Service: Handles alerts and updates
    - API Gateway: Provides a unified interface for external systems

4. **Monitoring & Management:**
    - Admin Dashboard: For configuration and monitoring
    - Monitoring & Alerting: Oversees system health and performance

### Benefits of the New Architecture:

1. Real-time processing: Enables immediate reactions to inventory changes and orders
2. Scalability: Each component can be scaled independently as needed
3. Resilience: Kafka's durability ensures data integrity even if individual services fail
4. Flexibility: Easy to add new services or modify existing ones
5. Observability: Centralized monitoring and management

This architecture addresses the overselling problem by enabling real-time inventory updates and order processing across all channels, while also providing a foundation for future growth and additional features.
