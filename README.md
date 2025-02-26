public class AdminActionService {
    @Autowired private ReportRepository reportRepository;

    public void actOnReport(Long reportId, ReportStatus newStatus) {
        Report report = reportRepository.findById(reportId)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Report not found"));
        
        report.setStatus(newStatus);
        reportRepository.save(report);
    }


@PreAuthorize("hasRole('ADMIN')")
public class AdminActionController {
    @Autowired private AdminActionService adminActionService;

    @PostMapping("/{reportId}/action")
    public ResponseEntity<String> actOnReport(@PathVariable Long reportId, @RequestParam ReportStatus status) {
        adminActionService.actOnReport(reportId, status);
        return ResponseEntity.ok("Report status updated to " + status);
    }
