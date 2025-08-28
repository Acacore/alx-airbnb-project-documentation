
# User Story: Manage Property Listings as a Host

## Title
As a registered host, I want to manage my property listings, including creating, editing, and deleting them, so that I can effectively control the properties I offer for rent and ensure they are accurately represented on the platform.

## Description
The host should have a dedicated interface within the Airbnb Clone System to manage their property listings. This includes the ability to create new listings with detailed information (e.g., location, price, availability), edit existing listings to update details or images, and delete listings that are no longer relevant. The system should provide real-time feedback and validation to ensure all inputs meet the platform's standards, and changes should be reflected immediately for guests to see.

## Acceptance Criteria
- **Given** I am a logged-in host, **when** I navigate to the "Manage Listings" section, **then** I should see a list of all my current property listings.
- **Given** I am on the "Manage Listings" page, **when** I click "Create New Listing," **then** I should be able to enter details such as property type, location, price, and availability, and upload photos.
- **Given** I submit a new listing with valid data, **when** I click "Save," **then** the listing should be added to my account and become visible to guests.
- **Given** I submit a new listing with invalid data (e.g., missing required fields), **when** I click "Save," **then** the system should display an error message and prevent submission.
- **Given** I select an existing listing, **when** I click "Edit," **then** I should be able to modify details and save the updated information.
- **Given** I select a listing, **when** I click "Delete," **then** the system should remove the listing from my account and notify me of the successful deletion.
- **Given** I make changes to a listing, **when** I save them, **then** the updated listing should be immediately available for guests to view.

