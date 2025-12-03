# Database Migrations

## Tool Choice

**Alembic is mandatory** for all PostgreSQL migrations. No exceptions.

## Configuration Requirements

### Transactional DDL

All migrations **MUST** run within transactions. Configure in `alembic.ini` or `env.py`:

```python
# env.py
def run_migrations_online():
    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
            transaction_per_migration=True,  # Each migration in its own transaction
        )
        with context.begin_transaction():
            context.run_migrations()
```

### Dry-Run by Default

Migrations **MUST** default to dry-run mode (execute then rollback):

```python
# Custom runner pattern
def run_migration(commit: bool = False) -> None:
    """Run migration with explicit commit flag.

    By default, runs migration in transaction then rolls back.
    Use --commit to persist changes.
    """
    with engine.begin() as conn:
        # Run the migration
        context.run_migrations()

        if commit:
            # Persist changes
            pass  # Transaction commits on exit
        else:
            # Show what would happen, then rollback
            print("DRY RUN: Changes rolled back. Use --commit to persist.")
            raise DryRunRollback()
```

CLI pattern:
```bash
alembic upgrade head           # Dry-run (default)
alembic upgrade head --commit  # Actually persist
```

## Migration Content Rules

### Reversibility

All migrations **SHOULD** be reversible with a working `downgrade()` function.

If a migration cannot be reversed:
1. Document this at design time
2. Add a warning in the migration file
3. Raise an error in `downgrade()`:

```python
def downgrade():
    raise IrreversibleMigrationError(
        "This migration drops the legacy_users table and cannot be reversed. "
        "Restore from backup if needed."
    )
```

### Data Migrations

Data migrations **MUST** be in the same file as related schema changes:

```python
def upgrade():
    # Schema change
    op.add_column('users', sa.Column('full_name', sa.String))

    # Data migration in same transaction
    connection = op.get_bind()
    connection.execute(
        text("UPDATE users SET full_name = first_name || ' ' || last_name")
    )

    # Remove old columns
    op.drop_column('users', 'first_name')
    op.drop_column('users', 'last_name')
```

**Never** split schema and data changes across multiple migrations for the same logical change.

## Migration Naming

Use descriptive names with timestamps:

```
2025_01_15_1423_add_user_email_verification.py
2025_01_16_0930_migrate_legacy_permissions.py
```
