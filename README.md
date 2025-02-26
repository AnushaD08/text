4️⃣ View User Groups API (/groups)
🔹 Group Entity
java
Copy
Edit
@Entity
@Table(name = "groups")
public class Group {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String name;

    @ManyToOne
    @JoinColumn(name = "creator_id", nullable = false)
    private User creator;

    @OneToMany(mappedBy = "group", cascade = CascadeType.ALL)
    private Set<GroupMembership> members;
}
🔹 Group Repository
java
Copy
Edit
@Repository
public interface GroupRepository extends JpaRepository<Group, Long> {
    List<Group> findByNameContainingIgnoreCase(String keyword);
}
🔹 Group Service
java
Copy
Edit
@Service
public class GroupService {
    @Autowired private GroupRepository groupRepository;

    public List<Group> getAllGroups(String search) {
        if (search != null && !search.isEmpty()) {
            return groupRepository.findByNameContainingIgnoreCase(search);
        }
        return groupRepository.findAll();
    }
}
🔹 Group Controller
java
Copy
Edit
@RestController
@RequestMapping("/groups")
public class GroupController {
    @Autowired private GroupService groupService;

    @GetMapping
    public ResponseEntity<List<Group>> getAllGroups(@RequestParam(required = false) String search) {
        return ResponseEntity.ok(groupService.getAllGroups(search));
    }
}
5️⃣ Create a Group API
java
Copy
Edit
@PostMapping
public ResponseEntity<Group> createGroup(@RequestParam String name, @RequestParam Long userId) {
    User user = userRepository.findById(userId)
        .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "User not found"));
    
    Group group = new Group();
    group.setName(name);
    group.setCreator(user);
    
    Group savedGroup = groupRepository.save(group);

    return ResponseEntity.status(HttpStatus.CREATED).body(savedGroup);
}
6️⃣ Add/Remove User to/from Group API
java
Copy
Edit
@PostMapping("/{groupId}/add/{userId}")
public ResponseEntity<String> addUserToGroup(@PathVariable Long groupId, @PathVariable Long userId) {
    groupService.addUserToGroup(groupId, userId);
    return ResponseEntity.ok("User added to group");
}

@DeleteMapping("/{groupId}/remove/{userId}")
public ResponseEntity<String> removeUserFromGroup(@PathVariable Long groupId, @PathVariable Long userId) {
    groupService.removeUserFromGroup(groupId, userId);
    return ResponseEntity.ok("User removed from group");
}
7️⃣ View User-Specific Groups API
java
Copy
Edit
@GetMapping("/user/{userId}")
public ResponseEntity<List<Group>> getUserGroups(@PathVariable Long userId) {
    return ResponseEntity.ok(groupService.getUserGroups(userId));
}
