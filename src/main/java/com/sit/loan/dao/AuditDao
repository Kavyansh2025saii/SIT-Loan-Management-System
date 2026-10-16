package com.sit.loan.dao;

import com.sit.loan.util.DBConnection;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

/** Writes the audit trail (e.g. user deletions) to the audit_log table. */
public class AuditDao {
    public void log(String action, String performedBy, Integer targetUserId, String targetUsername, String details) {
        String sql = "INSERT INTO audit_log (action, performed_by, target_user_id, target_username, details) VALUES (?,?,?,?,?)";
        try (Connection c = DBConnection.get(); PreparedStatement ps = c.prepareStatement(sql)) {
            ps.setString(1, action);
            ps.setString(2, performedBy);
            if (targetUserId == null) ps.setNull(3, java.sql.Types.INTEGER); else ps.setInt(3, targetUserId);
            ps.setString(4, targetUsername);
            ps.setString(5, details);
            ps.executeUpdate();
        } catch (SQLException e) {
            System.out.println("[WARN] Could not write audit log: " + e.getMessage());
        }
    }
}
