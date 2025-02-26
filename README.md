@Service
public class GroupService {
    
    @Autowired
    private GroupRepository groupRepository;
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private GroupMembershipRepository groupMembershipRepository;

    /**
     * Adds a user to a group.
     */
    public void addUserToGroup(Long groupId, Long userId) {
        Group group = groupRepository.findById(groupId)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Group not found"));

        User user = userRepository.findById(userId)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "User not found"));

        // Check if user is already a member
        if (groupMembershipRepository.existsByUserAndGroup(user, group)) {
            throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "User is already a member of the group");
        }

        // Add user to the group
        GroupMembership membership = new GroupMembership();
        membership.setUser(user);
        membership.setGroup(group);
        groupMembershipRepository.save(membership);
    }

    /**
     * Removes a user from a group.
     */
    public void removeUserFromGroup(Long groupId, Long userId) {
        Group group = groupRepository.findById(groupId)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Group not found"));

        User user = userRepository.findById(userId)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "User not found"));

        // Check if user is a member of the group
        GroupMembership membership = groupMembershipRepository.findByUserAndGroup(user, group)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.BAD_REQUEST, "User is not a member of the group"));

        // Remove user from the group
        groupMembershipRepository.delete(membership);
    }
}





@Repository
public interface GroupMembershipRepository extends JpaRepository<GroupMembership, Long> {

    boolean existsByUserAndGroup(User user, Group group);

    Optional<GroupMembership> findByUserAndGroup(User user, Group group);

    void deleteByUserAndGroup(User user, Group group);
}
