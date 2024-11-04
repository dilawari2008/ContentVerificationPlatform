# Reactive Network Hackathon - Content Verification Platform

_Note - This repo just has the approach and not the dapp implementation. Any code snippet should be treated as a pseudo code snippet._


### The Idea

A lot of times content verification needs to be done to check if the content being uploaded adheres to the regulations of the platform. So basically we have 2 sections here
1) content upload and listing
2) content verification

Using Reactive Smart Contracts (RSC's) these two modules can be isolated and used separately, so that we can plug and play with the different combinations of each module, i.e. a listing module can connect to a different verification module if it finds the current verification module unsatisfactory, and a verification module can be used by several listing modules. This also keeps the verification process asynchronous.

### Scenario

Lets take a simple example of an NFT Marketplace for the demo of the approach. This NFT Marketplace is a travel related nft marketplace, i.e. the content should be travel related. Users can list their nft's on the marketplace which can have photographs or essays describing their travel experience. When they list, the nft's metadata needs to be extracted and parsed to extract all urls and texts. Now we need to verify whether these urls and texts are relevant to the domain of travel.
But too many manual verifications may take a lot of time. So we can use some automated method for verification and trust its results. If the user finds that the verification is wrong, they can challenge the verification and for the challenges registered, we can carry out manual verification.
Once the verification is successful, the listing platform can show the listing on the dashboard. In this way, we will be able to filter out any nft of other domains (such as art, music) that might have been attempted to be listed on the platform.


### Implementation

_Reminder - This repo just has the approach and not the dapp implementation. Any code snippet should be treated as a pseudo code snippet._

<img width="896" alt="Screenshot 2024-11-04 at 6 11 17 AM" src="https://github.com/user-attachments/assets/219ab471-add9-4607-8a4e-9d1216ba6055">

#### Explanation
1. The user attempts to list their nft on the listing dapp, which gets registered in the contract but not visible on the dashboard.
2. The nft details are stored in a mapping with initial values of verification data as attemps 0, status InProgress and isMaxVerificationDone false.
3. After registering the details, the metadata is extracted and sent to the RSC1 for verification to proceed.
4. Then an event is emitted with the nft data and the verification data, this event is indexed by the listing dapp to show the user their nft's verification status.
5. This event is subscribed to by the RSC1 and the RSC1 calls the attempVerificationCallback method on the VerificationContract.
6. Based on the number of attempts and isMaxVerificationDone, the verification contract decides the VerificationMethod to be carried out.
   if (isMaxVerificationDone == true) return with the current verification details
   else if (attempts == 0) Ai
   else if (attempts == 1) Manual
   else SubjectMatterExpert
7. Once decided, this object is inserted into a map against is current verification method, and an event is emitted with the method and nftMetadata details.
8. This event is indexed by the verification dapp and can then be categorised into the categories on the ui, where MANUAL and SubjectMatterExpert jobs can be picked by the respective owners and the Ai verification keeps running by Chainlink Keepers in bathces in regular time intervals, calling an Ai verification method off chain through Chainlink External Adapters.
9. Once the verification process is complete, the mapping is updated and event is emitted with the nftData and the verification results.
10. This event is subscribed to by the RSC2 and calls the verificationStatusUpdateCallback on the ListingContract.
11. The ListingContract then updates the mapping and emits the event with the new verification status, which is indexed by the listing dapp.
12. If the new status is Verified, the nft is visible on the dashboard.
13. If it is not verified and the user feels there is a mistake, they can challenge, in which the verification process will repeat but with one verification method up in the heirarchy (Ai -> Manual -> SubjectMatterExpert).
14. If at any point isMaxVerificationDone is true, the user cannot challenge.


### Changing the modules
If someone wants a new Verification Module, they can change the subscriber in RSC2 and callback in RSC1 to point to the new Verification Module.

If an org has created a Verification Module as a service and wants to expose it to the world as a stand alone service, the users can connect to it using their own RSC's. The users will also have to pass an id for their listing dapp, which will then be filtered by the RSC2 in the event received and then the callback shall be called on the listing service.


