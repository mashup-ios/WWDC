# [Drag and Drop with Collection and Table View](https://developer.apple.com/videos/play/wwdc2017/223/)

@ WWDC 17

어제 봤던 Introducing Drag and Drop의 실전편이 아닐까 기대하면서 봅니당 호홓



### Drag and Drop Delegates

dragDelegate와 dropDelegate는 서로에게 독립적이다.



### Beginning a Drag

* One required method on `dragDelegate`

  ```swift
  func collectionView(_: UICollectionView, itemsForBeginning: UIDragSession, at: IndexPath) -> [UIDragItem]
  ```

* Return an empty array to prevent the drag



### Adding Items to a Drag Session

drag 하는 건 굉장히 쉽다~!!

* Opt-in via optional method on `dragDelegate`

  ```swift
  func collectionView(_: UICollectionView, itemsForAddingTo: UIDragSession, at: IndexPath, point: CGPoint) -> [UIDragItem]
  ```

* Return an empty array to handle tap normally



### Acceptinig a Drop

* One required method on `dropDelegate`

  ```swift
  func collectionView(_: UICollectionView, performDropWith: UICollectionViewDropCoordinate)
  ```

* Drop coordinator

  * Access dropped items
  * Update collection/table view
  * Specify animations



### Drop Proposal

* How you want to handle the drop
  * 사용자가 터치를 멈추고 drop이 수행되기 전 drop session을 어떻게 다룰 건지 시스템에게 알려주는 단계

* operation
  * `.copy`, `.move`, `.cancel`, `.forbidden`
* intent



### Drop Intent

* Additional information for collection and table view
* `.unspecified`: 아무 효과 없음
* `.insertAtDestinationIndexPath`: 들어갈 indexpath의 영역을 비워줌!
* `.insertIntoDestinationIndexPath`: 들어갈 자리를 보여줌!

* `.automatic`: 위의 두개 짬뽕쓰 느낌



```swift
// Providing a Drop Proposal
func collectionView(_ collectionView: UICollectionView, dropSessionDidUpdate session: UIDropSession, withDestinationIndexPath destinationIndexPath: IndexPath?) -> UICollectionViewDropProposal {
  if session.localDragSession != nil {
    return UICollectionViewDropProposal(operation: .move, intent: .insertAtDestinationIndexPath)
  } else {
    return UICollectionViewDropProposal(operation: .copy, intent: .insertAtDestinationIndexPath)
  }
}
```



### Drop Animations

* Set up animations using the drop coordinator

```swift
// Drop to a Newly Inserted Item
func collectionView(_ collectonView: UICollectionView, perfrmDropWith coordinator: UICollectionViewDropCoordiinator) {
	guard let destinationIndexPath = coordinator.destinationIndexPath,
  	let dragItem = coordinator.items.first?.dragItem,
  	let image = dragItem.localObject as? UIImage else { return }
  
  collectionView.performBatchUpdates({
    self.imagesArray.insert(image, at: destinationIndexPath.item)
    collectionView.insertItems(at: [destinationIndexPath])
  })
  
  coordinator.drop(dragItem, toItemAt: destinationIndexPath)
}
```

* Drop into an item/row

```swift
// Drop Into a Row
func tableViiew(_ tableView: UITableView, performDropWith coordinator: UITableViewDropCoordinator) {
  guard let destinationIndexPath = coordinator.destinationIndexPath,
  	let dragItem = coordiinator.items.first?.dragItem,
  	let image = dragItem.localObject as? UIImage else { return }
  
  let photoAlbumIndex = destinationIndexPath.row
  guard photoAlbumIndex < self.albumsArray.count else { return }
  self.add(image: imagee, toAlbumAt: photoAlbumIndex)
  
  if let cell = tableView.cellForRow(at: destinationIndexPath), let view = cell.imageView {
    let rect = cell.convert(view.bounds, from: view)
    coordinator.drop(dragItem, intoRowAt: destinationIndexPath, rect: rect)
  }
}
```



### Animating Items Before the Data Loads

* Data loads asynchronously
* Bookkeeping is difficult



### Placeholders

* Temporary insertiions in the collection/table view
* You provide the cell, we do the bookkeeping for you
* Great experience while data loads asynchronously

```swift
// Drop to a Placeholder Item
func collectioinView(_ collectionView: UICollectionView, performDropWith coordinator: UICollectionViewDropCoordinator) {
  
  guard let destinationIndexPath = coordinator.destinationIndexPath else { return }
  
  for item in coordinator.items {
    let placeholderContext = coordinator.drop(item.dragItem, toPlaceholderInsertedAt: destinationIndexPath, withReuseIdentifier: "PlaceholderCell") { cell in
      // Configure the placeholder cell
    }
  }
}
```



### Using the Placeholder Context

* Commit the insertion of a placeholder to exchange it for the final cell
* Delete the placeholder if it's no longer needed

```swift
// Using the Placeholder Context
for item in coordinator.items {
  let placeholderContext = /* insert the placeholder for this item */
  
  item.dragItem.itemProvider.loadObject(ofClass: UIImage.self) { (object, error) in
    DispatchQueue.main.async {
      if let image = object as? UIImage {
        placeholderContext.commitInsertion { insertionIndexPath in
          self.imagesArray.insert(image, at: insertionIndexPath.item)
        }
      } else {
        placeholderContext.deletePlaceholder()
      }
    }
  }
}
```



### Working with Placeholders

* Avoid `reloadData`, use `performBatchUpdates(_:completion:)` instead

* When placeholders exist, the table or collection view has uncommitted updates

  ```swift
  var hasUncommitedUpdates: Bool { get }
  ```

  

### Supporting Reordering

* Implement `dropDelegate` method

```swift
func collectionView(_: UICollectionView, dropSessionDidUpdate: UIDropSession, withDestinatioonIndexPath: IndexPath?) -> UIColectionViewDropProposal
```

* Return a drop proposal

```swift
return UICollectionViewDropProposal(opperation: .move, intent: .insertatDestinationIndexPath)
```

* Coontinue to implement the existing data source method

```swift
func tableView(_: UITableView, moveRowAt: IndexPath, to: IndexPath)
```

* Called instead of `tableView(_: UITableView, pperformDropWith: UITableViewDropCoordinator)`
* Provide a `dragDelegate` and `dropDelegate`
* Inside `collectionView(_: UICollectionView, performDropWith: UICollectionViewDropCoordinator)`
  * Delete from `UICollectionViewDropItem.sourceIndexPath`
  * Insert at `UICollectionViewDropCoordinator.destinationIndexPath`



### Spring Loading

* Table and collection view conform to `UISpringLoadedInteractionSupporting`

```swift
var isSpringLoaded: Bool { get set }
```

* Customize spring loading with the optional delegate method

```swift
func collectionView(_: UICollectionView, shouldSpringLoadItemAt: IndexPath, with: UISpriingLoadedInteractionConotext) -> Bool
```



### Customizing the Drag Preview

* By default, the entire cell is used as the drag preview
* Provde drag preview parameters by implementing `dragDelegate` method

```swift
func collectionView(_: UICollectionView, dragPreviewParametersForItemAt: IndexPath) -> UIDragPreviewParameters?
```

