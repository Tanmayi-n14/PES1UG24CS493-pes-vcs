README INCLUDES PHASE 5 AND 6 AND SCREENSHOTS FROM PHASE 1 TO 4

## 🔍 Phase 5: Branching and Checkout Analysis

### **Q5.1: Implementation of `pes checkout <branch>`**
* **Files Changed:** The `.pes/HEAD` file must be updated to point to the new branch reference (e.g., changing from `ref: refs/heads/main` to `ref: refs/heads/feature-branch`).
* **Working Directory:** The current files in the working directory must be deleted or overwritten to match the tree structure of the target commit. 
* **Complexity:** The operation is complex because it must be safe and atomic. It must handle "unsaved" changes—if a user has modified a file that differs between the current and target branches, the operation must be aborted to prevent data loss. Additionally, it requires efficient recursive traversal of tree objects to reconstruct the directory structure.

### **Q5.2: Detecting a "Dirty" Working Directory**
* **Step 1:** Compare the working directory files with the **Index**. If the file's metadata (timestamp/size) or hash differs from the index entry, the file is "dirty" with unstaged changes.
* **Step 2:** Compare the **Index** hash with the hash stored in the **Current Commit's Tree**. If they differ, there are staged but uncommitted changes.
* **Conflict Detection:** If either check reveals a "dirty" state, we then compare the current version of that file with the version in the **Target Branch's Tree**. If the target version is different from the current version, a conflict is detected, and the checkout is refused.

### **Q5.3: The "Detached HEAD" State**
* **Commits:** Commits proceed normally, and each new commit's `parent` will point to the previous hash. However, since no branch reference (like `main`) is being updated, these commits are not part of any named timeline.
* **Recovery:** Once the user checks out another branch, the "detached" commits become orphaned. A user can recover them by finding the specific commit hash (using a tool like `reflog`) and creating a new branch at that hash using `pes branch <name> <hash>`.

---

## 🧹 Phase 6: Garbage Collection Analysis

### **Q6.1: Reachability Algorithm**
* **Algorithm (Mark and Sweep):** 1. **Mark:** Start at all "roots" (all branch refs in `refs/heads/`, the `HEAD` file, and tags). Recursively walk through every commit to find its parent and its root tree. From each tree, identify all sub-trees and blobs. 
    2. **Sweep:** Compare the list of "reachable" hashes against all files in `.pes/objects`. Delete any file whose hash was not marked.
* **Data Structure:** A **Hash Set** (or a Bloom Filter for very large scales) is used to store the 64-character hex strings of reachable objects to ensure $O(1)$ lookup and avoid redundant processing.
* **Estimation:** You would visit at least 100,000 commit objects. Since most commits share the majority of their files, the total number of unique visits would likely range between **300,000 to 700,000 objects** to map reachability.

### **Q6.2: GC Race Conditions**
* **The Danger:** A race condition occurs when a new process writes a blob or tree to the object store but hasn't yet linked it to a commit object or branch. 
* **Race Condition:** 1. Process A (Commit) writes a new Blob `H1`. 
    2. Process B (GC) starts, sees `H1` in the objects folder but finds no branch/commit pointing to it yet. 
    3. GC deletes `H1`. 
    4. Process A finishes and creates a Commit pointing to `H1`, but `H1` is now gone, leaving the repository corrupted.
* **Git's Solution:** Git GC uses a **grace period** (typically 2 weeks). It only prunes objects that are both unreachable **and** older than a specific threshold. This ensures that "in-flight" objects are protected during the commit process.

---

## 📸 Implementation Screenshots

### Phase 1: Object Store
- **Screenshot 1A:** <img width="1920" height="921" alt="VirtualBox_Ubuntu 24 04 3_20_04_2026_15_43_40" src="https://github.com/user-attachments/assets/45dc5324-9008-45eb-8669-60b87fe7ff82" />
- **Screenshot 1B:** <img width="1920" height="921" alt="VirtualBox_Ubuntu 24 04 3_20_04_2026_15_44_18" src="https://github.com/user-attachments/assets/4d05c93d-29fd-4b03-97b1-2a243bef3607" />



### Phase 2: Tree Objects
- **Screenshot 2A (Tests Passing):** <img width="1920" height="921" alt="VirtualBox_Ubuntu 24 04 3_20_04_2026_16_02_33" src="https://github.com/user-attachments/assets/bf42fffb-0b5c-4df1-b256-62beb95bc66c" />

- **Screenshot 2B (Binary Format):** <img width="1920" height="921" alt="VirtualBox_Ubuntu 24 04 3_20_04_2026_16_26_38" src="https://github.com/user-attachments/assets/b0075162-70cb-4c95-9822-cd2ad8e8ee3e" />


### Phase 3: The Index
- **Screenshot 3A (Staged Changes):** <img width="1920" height="921" alt="VirtualBox_Ubuntu 24 04 3_20_04_2026_16_20_16" src="https://github.com/user-attachments/assets/ddb21a74-55ed-4ab8-a696-521d43a7c31c" />

- **Screenshot 3B (Index File):** <img width="1920" height="921" alt="VirtualBox_Ubuntu 24 04 3_20_04_2026_16_21_06" src="https://github.com/user-attachments/assets/7aeba0e8-3ea0-482b-bb1d-f700feb559bf" />


### Phase 4: Commits
- **Screenshot 4A (Commit Log):** <img width="1920" height="921" alt="VirtualBox_Ubuntu 24 04 3_20_04_2026_16_56_33" src="https://github.com/user-attachments/assets/4ad33ecf-2d47-4c07-8321-5be504c60a06" />

- **Screenshot 4B (Object Growth):** <img width="1920" height="921" alt="VirtualBox_Ubuntu 24 04 3_20_04_2026_16_57_14" src="https://github.com/user-attachments/assets/eed851fd-5cfe-4e51-a2f0-ee51d300f685" />

- **Screenshot 4C (Ref Chain):** <img width="1920" height="921" alt="VirtualBox_Ubuntu 24 04 3_20_04_2026_16_58_54" src="https://github.com/user-attachments/assets/5a714578-f924-4c18-83fb-a73d3078a1c3" />



