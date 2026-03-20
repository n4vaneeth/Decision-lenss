from utils.normalization import normalize_key

# Captures multiple lines of input until 'done' is typed.
def get_multiline_input(prompt: str, end_token: str = "done") -> str:
    print(prompt)
    print(f"Type '{end_token}' on a new line when finished.\n")

    lines = []

    while True:
        line = input("> ")
        if line.strip().lower() == end_token:
            break
        lines.append(line)

    return "\n".join(lines)

# Gathers criteria and their benefit/cost classification.
def get_criteria_from_user(prompt):
    criteria = []
    criteria_types = {}

    print(prompt)
    print("Type 'done' when finished.\n")

    while True:
        criterion = input("> ").strip()
        if criterion.lower() == "done":
            break
        if not criterion:
            continue

        criteria.append(criterion)

        while True:
            ctype = input(
                f'Is "{criterion}" a benefit or cost? (b = benefit, c = cost): '
            ).lower()

            if ctype == "b":
                criteria_types[criterion] = "benefit"
                break
            elif ctype == "c":
                criteria_types[criterion] = "cost"
                break
            else:
                print("Please enter 'b' or 'c'.")

    return criteria, criteria_types


# Calculates importance weights from a user-provided ranking.
def get_weights_from_ranking(criteria):
    rankings = {}
    used_ranks = set()

    print("\nRank criteria by importance (1 = highest priority).")
    print("Each rank must be UNIQUE (no ties).\n")

    for criterion in criteria:
        while True:
            try:
                rank = int(input(f"{criterion}: "))

                if not (1 <= rank <= len(criteria)):
                    print(f"Enter a number between 1 and {len(criteria)}.")
                    continue

                if rank in used_ranks:
                    print(
                        f"Rank {rank} already used. "
                        "Each criterion must have a unique rank."
                    )
                    continue

                rankings[criterion] = rank
                used_ranks.add(rank)
                break

            except ValueError:
                print("Please enter a valid integer.")

    max_rank = len(criteria)

    raw_weights = {
        c: (max_rank + 1) - r
        for c, r in rankings.items()
    }

    total = sum(raw_weights.values())
    weights = {c: w / total for c, w in raw_weights.items()}

    return weights

def confirm_criteria_types(criteria):
    """
    Allows user to confirm or toggle cost/benefit classification.
    Reprints state after every change for clarity.
    """

    while True:
        print("\nConfirm criteria types:")
        keys = list(criteria.keys())

        for i, c in enumerate(keys, start=1):
            print(f"{i}. {c} → {criteria[c]['type']}")
        print("\nType numbers to toggle, 'done' when satisfied.")
        choice = input(
            "\nEnter criterion number to toggle (or 'done'): "
        ).strip().lower()

        if choice == "done":
            break

        if not choice.isdigit():
            print("Please enter a valid number or 'done'.")
            continue

        idx = int(choice) - 1
        if idx < 0 or idx >= len(keys):
            print("Invalid choice.")
            continue

        key = keys[idx]
        current = criteria[key]["type"]
        criteria[key]["type"] = "benefit" if current == "cost" else "cost"

        print(f"\n✔ {key} set to {criteria[key]['type']}")

    return criteria

def confirm_ordinal_scales(criteria: dict, options: dict) -> dict:
    """
    Allows the user to confirm or edit ordinal scales.
    Distinguishes between USER-defined and AUTO-generated scales.
    """

    # ---------------- helper: auto-generate simple scales ----------------
    def auto_generate_scale(criterion_name: str):
        name = criterion_name.lower()

        if "salary" in name or "price" in name:
            return ["low", "medium", "high"]
        if "clearance" in name:
            return ["low", "adequate", "high"]
        if "safety" in name:
            return ["poor", "average", "good"]
        if "comfort" in name:
            return ["poor", "okay", "good", "excellent"]
        if "feel" in name or "driving" in name:
            return ["boring", "decent", "fun"]

        return ["low", "medium", "high"]
    # --------------------------------------------------------------------

    for raw_key, meta in criteria.items():

        # Skip explicitly ignored criteria
        if meta.get("ignored"):
            continue

        key = normalize_key(raw_key)
        scale = meta.get("scale")

        # ---------- detect numeric data ----------
        has_numeric = False
        for opt_vals in options.values():
            val = opt_vals.get(raw_key) or opt_vals.get(key)
            if isinstance(val, (int, float)):
                has_numeric = True
                break
        # ----------------------------------------

        # ---------- scale missing & no numeric ----------
        if not scale and not has_numeric:
            print(f"\nCriterion '{raw_key}' has no numeric values.")
            print("How would you like to handle this?")
            print("[1] Define an ordinal scale (recommended)")
            print("[2] Auto-generate a simple scale for me")
            print("[3] Ignore this criterion")

            choice = input("Choice (default = 1): ").strip() or "1"

            if choice == "1":
                user_scale = input(
                    f"Enter ordinal scale for '{raw_key}' "
                    "(comma-separated, lowest → highest): "
                ).strip()

                if not user_scale:
                    print(f"✔ '{raw_key}' ignored.")
                    meta["ignored"] = True
                    continue

                meta["scale"] = [s.strip() for s in user_scale.split(",")]
                meta["scale_source"] = "user"
                scale = meta["scale"]

            elif choice == "2":
                auto_scale = auto_generate_scale(raw_key)
                meta["scale"] = auto_scale
                meta["scale_source"] = "auto"
                scale = auto_scale

                print(
                    f"✔ Auto-generated scale for '{raw_key}': "
                    f"{' < '.join(auto_scale)}"
                )

                #if this was numeric before, allow value entry ONCE
                if meta.get("unit") and not has_numeric:
                    meta["_needs_value_entry"] = True
            else:
                print(f"✔ '{raw_key}' ignored.")
                meta["ignored"] = True
                continue
        # --------------------------------------------

        # Numeric-only criteria → skip
        if not scale:
            continue

        # ---------- confirm scale ----------
        print("\nCriterion:", raw_key)
        print("Proposed order (worst → best):")
        for i, v in enumerate(scale, start=1):
            print(f"{i}. {v}")

        confirm = input("Is this order correct for you? (y/n): ").strip().lower()

        if confirm != "y":
            edited = input(
                "Enter corrected scale (comma-separated, lowest → highest): "
            ).strip()

            if edited:
                meta["scale"] = [s.strip() for s in edited.split(",")]
                meta["scale_source"] = "user"
                scale = meta["scale"]
        # -----------------------------------

        # ---------- detect descriptor mismatches ----------
        detected = set()
        for opt_vals in options.values():
            val = opt_vals.get(key)
            if isinstance(val, str) and normalize_key(val) != "unknown":
                detected.add(normalize_key(val))

        scale_norm = {normalize_key(s) for s in scale}
        missing = detected - scale_norm
        # --------------------------------------------------

        if missing:
            updated_scale, alias_map, ignored_vals = resolve_scale_mismatch(
                criterion_name=raw_key,
                scale=scale,
                unknown_values=missing
            )

            meta["scale"] = updated_scale

            if alias_map:
                meta.setdefault("aliases", {}).update(alias_map)

            if ignored_vals:
                meta.setdefault("ignored_values", set()).update(ignored_vals)

        # Numeric → auto ordinal: allow ONE value-entry pass
        if meta.get("_needs_value_entry"):
            from utils.input_helpers import prompt_missing_values_for_criterion

            prompt_missing_values_for_criterion(
                criterion=raw_key,
                criteria=criteria,
                options=options
            )

            meta.pop("_needs_value_entry", None)
    return criteria

# Manages addition and removal of decision options.
def confirm_options(options: dict, criteria: dict) -> dict:
    while True:
        print("\nIdentified options:")
        keys = list(options.keys())

        for i, opt in enumerate(keys, start=1):
            print(f"{i}. {opt}")

        choice = input(
            "\n[a] Add option  [r] Remove option  [c] Continue: "
        ).strip().lower()

        if choice == "c":
            break

        elif choice == "a":
            name = input("Enter new option name: ").strip()
            if not name:
                continue

            options[name] = {}
            print(f"\nEnter attributes for {name} (press Enter to skip):")

            for raw_c, meta in criteria.items():
                c = normalize_key(raw_c)
                scale = meta.get("scale")
                unit = meta.get("unit")

                # Build clear prompt
                if scale:
                    prompt = f"  {raw_c} (choose from: {', '.join(scale)}): "
                else:
                    unit_label = f", unit: {unit}" if unit else ""
                    prompt = f"  {raw_c} (enter number only{unit_label}): "

                while True:
                    val = input(prompt).strip()

                    # Allow skip
                    if not val:
                        break

                    # Ordinal criterion
                    if scale:
                        norm_val = normalize_key(val)
                        scale_norm = [normalize_key(s) for s in scale]

                        if norm_val not in scale_norm:
                            print(
                                f"  Invalid value. Allowed: {', '.join(scale)}"
                            )
                            continue

                        options[name][c] = val
                        break

                    # Numeric criterion
                    else:
                        try:
                            options[name][c] = float(val)
                            break
                        except ValueError:
                            print(
                                "  Invalid value. Please enter a number only."
                            )

        elif choice == "r":
            idx = input("Enter option number to remove: ").strip()
            if idx.isdigit():
                i = int(idx) - 1
                if 0 <= i < len(keys):
                    options.pop(keys[i])

    return options

def confirm_criteria(criteria: dict, options: dict) -> dict:
    """
    Allows the user to add or remove criteria.
    If a new criterion is added:
    - numeric values are requested ONLY if a unit is provided
    - otherwise, value entry is deferred to ordinal scale handling
    """

    while True:
        print("\nIdentified criteria:")
        keys = list(criteria.keys())

        for i, k in enumerate(keys, start=1):
            ctype = criteria[k].get("type", "unknown")
            print(f"{i}. {k} ({ctype})")

        choice = input(
            "\n[a] Add criterion  [r] Remove criterion  [c] Continue: "
        ).strip().lower()

        if choice == "c":
            break

        elif choice == "a":
            name = input("Enter criterion name: ").strip()
            if not name:
                continue

            key = normalize_key(name)

            ctype = input(
                "Is this a benefit or cost? (b = benefit, c = cost): "
            ).strip().lower()

            unit = input(
                "Enter unit (or press Enter if not applicable): "
            ).strip()

            criteria[key] = {
                "type": "benefit" if ctype == "b" else "cost",
                "description": "User-added criterion",
                "scale": [],
                "unit": unit if unit else None
            }

            # Ask for numeric values ONLY if a unit exists
            if unit:
                print(f"\nEnter values for new criterion '{name}' (press Enter to skip):")

                for opt, attrs in options.items():
                    prompt = f"  {opt} (enter number only, unit: {unit}): "

                    while True:
                        val = input(prompt).strip()
                        if not val:
                            break
                        try:
                            attrs[key] = float(val)
                            break
                        except ValueError:
                            print("  Invalid number. Please enter a numeric value.")
            else:
                # No unit → defer value entry
                print(
                    f"\nℹ Skipping value entry for '{name}' — "
                    "you’ll define a scale next if needed."
                )

        elif choice == "r":
            idx = input("Enter criterion number to remove: ").strip()
            if idx.isdigit():
                i = int(idx) - 1
                if 0 <= i < len(keys):
                    criteria.pop(keys[i])

    return criteria

def resolve_scale_mismatch(
    criterion_name: str,
    scale: list[str],
    unknown_values: set[str]
) -> tuple[list[str], dict[str, str], set[str]]:
    """
     Returns:
     - updated_scale
     - alias_map (e.g. {"mostly_good": "okay"})
     - ignored_values
     """

    alias_map = {}
    ignored = set()
     # Helper: suggest mappings by relative position / similarity
    def suggest_mappings(values, scale):
        suggestions = {}
        for v in values:
            v_norm = v.replace("_", " ")
            if "extreme" in v_norm or "excellent" in v_norm:
                suggestions[v] = scale[-1]
            elif "very" in v_norm or "good" in v_norm:
                suggestions[v] = scale[-2] if len(scale) >= 2 else scale[-1]
            elif "decent" in v_norm or "okay" in v_norm:
                suggestions[v] = scale[len(scale)//2]
            elif "not" in v_norm or "poor" in v_norm:
                suggestions[v] = scale[0]
        return suggestions

    print(
        f"\n⚠ The following values are not in the scale for '{criterion_name}': "
        f"{', '.join(unknown_values)}"
    )
    print("You will resolve them one by one.\n")

    for val in unknown_values:
        print(f'\nResolving value: "{val}"')

        while True:
            choice = input(
                "\nChoose how to resolve this:\n"
                "[a] Add to scale\n"
                "[m] Map to existing value\n"
                "[i] Ignore this value\n"
                "[e] Edit entire scale\n"
                "Choice: "
            ).strip().lower()

            # -------- ADD TO SCALE ----------
            if choice == "a":
                print_scale(scale)

                pos = input(
                    f"Insert '{val}' at position (1–{len(scale)+1}): "
                ).strip()

                if pos.isdigit():
                    idx = int(pos) - 1
                    if 0 <= idx <= len(scale):
                        scale.insert(idx, val)
                        print(f'\n✔ Added "{val}" to scale.')
                        print_scale(scale)   # ✅ VISUAL FEEDBACK
                        break

            # ---------- MAP ----------
            elif choice == "m":
                print("\nMap to:")
                for i, s in enumerate(scale, start=1):
                    print(f"{i}. {s}")

                sel = input("Choose number: ").strip()
                if sel.isdigit():
                    idx = int(sel) - 1
                    if 0 <= idx < len(scale):
                        alias_map[val] = scale[idx]
                        print(f'\n✔ Mapped "{val}" → "{scale[idx]}"')
                        print_scale(scale)   # ✅ SHOW CURRENT STATE
                        break

            # ---------- IGNORE ----------
            elif choice == "i":
                remaining = unknown_values - ignored - {val}

                if not remaining:
                    print(
                        f"\n⚠ WARNING:\n"
                        f"Ignoring this value will leave NO usable data for "
                        f'"{criterion_name}".'
                    )
                else:
                    print(
                        f"\n⚠ Warning: Ignoring this value will remove "
                        f"usable data for some options under "
                        f'"{criterion_name}".'
                    )

                if input("Proceed anyway? (y/n): ").strip().lower() == "y":
                    ignored.add(val)
                    print(f'\n✔ Ignored "{val}".')
                    print_scale(scale)  
                    break

            # ---------- EDIT ENTIRE SCALE ----------
            elif choice == "e":
                new_scale = input(
                    "Enter new scale (comma-separated, lowest → highest): "
                ).strip()
                if not new_scale:
                    continue

                scale[:] = [s.strip() for s in new_scale.split(",")]
                print(f"\n✔ Replaced entire scale for '{criterion_name}'.")
                print_scale(scale)   # SHOW NEW BASELINE

                followup = input(
                    "\nHandle existing values:\n"
                    "[a] Map manually\n"
                    "[m] Auto-map similar values\n"
                    "[i] Ignore all\n"
                    "Choice: "
                ).strip().lower()

                if followup == "m":
                    suggestions = suggest_mappings(unknown_values, scale)
                    print("\nSuggested mappings:")
                    for k, v in suggestions.items():
                        print(f"{k} → {v}")

                    if input("Apply these mappings? (y/n): ").strip().lower() == "y":
                        alias_map.update(suggestions)
                        print("\n✔ Auto-mappings applied.")
                        print_scale(scale)
                    break

                elif followup == "a":
                    for v in unknown_values:
                        print(f"\nMap '{v}' to:")
                        for i, s in enumerate(scale, start=1):
                            print(f"{i}. {s}")
                        sel = input("Choose number (or Enter to skip): ").strip()
                        if sel.isdigit():
                            idx = int(sel) - 1
                            if 0 <= idx < len(scale):
                                alias_map[v] = scale[idx]
                                print(f'✔ Mapped "{v}" → "{scale[idx]}"')
                    print_scale(scale)
                    break

                elif followup == "i":
                    print(
                        f"\n⚠ WARNING: All values ignored for '{criterion_name}'."
                    )
                    if input("Proceed anyway? (y/n): ").strip().lower() == "y":
                        ignored.update(unknown_values)
                        print_scale(scale)
                    break

    return scale, alias_map, ignored


# Prints the current state of an ordinal scale.
def print_scale(scale):
    print("\nUpdated scale (lowest → highest):")
    for i, s in enumerate(scale, start=1):
        print(f"{i}. {s}")


# Navigation menu shown after viewing results.
def post_result_menu():
    print("\nWhat would you like to do next?")
    print("[1] Change criteria importance")
    print("[2] Add / remove / edit options")
    print("[3] Add / remove / edit criteria")
    print("[0] Exit")

    return input("Choice: ").strip()

from utils.normalization import normalize_key

def resolve_empty_scaled_criteria(criteria: dict, options: dict) -> dict:
    """
    Handles USER-defined ordinal criteria that have a scale
    but no option values.

    AUTO-generated scales are NEVER prompted here.
    """

    for raw_criterion, meta in list(criteria.items()):

        # Skip ignored criteria
        if meta.get("ignored"):
            continue

        # Skip criteria without a scale
        if not meta.get("scale"):
            continue

        # CRITICAL FIX:
        # Only USER-defined scales are allowed to prompt here
        if meta.get("scale_source") != "user":
            continue

        key = normalize_key(raw_criterion)

        # Check if ALL options are missing this criterion
        all_missing = all(
            options.get(opt, {}).get(key) in (None, "unknown")
            for opt in options
        )

        if not all_missing:
            continue

        print(
            f"\n⚠ You defined a scale for '{raw_criterion}', "
            f"but no options have values for it."
        )
        print("Would you like to:")
        print("[1] Enter values now")
        print("[2] Ignore this criterion")

        choice = input("Choice (default = 2): ").strip() or "2"

        if choice == "1":
            from utils.input_helpers import prompt_missing_values_for_criterion
            prompt_missing_values_for_criterion(
                criterion=raw_criterion,
                criteria=criteria,
                options=options
            )
        else:
            meta["ignored"] = True
            print(f"✔ '{raw_criterion}' has been ignored.")

    return criteria

# Prompts user to enter missing values for a specific criterion.
def prompt_missing_values_for_criterion(
    criterion: str,
    criteria: dict,
    options: dict
):
    meta = criteria[criterion]
    scale = meta.get("scale")
    unit = meta.get("unit")

    print(f"\nEnter values for '{criterion}':")

    for opt in options:
        if options[opt].get(criterion) not in (None, "unknown"):
            overwrite = input(
                f"  {opt} already has a value. Overwrite? (y/n): "
            ).strip().lower()
            if overwrite != "y":
                continue

        if scale:
            # Ordinal value: force explicit choice
            while True:
                val = input(
                    f"  {opt} (choose from: {', '.join(scale)} or type 'skip'): "
                ).strip()

                if not val:
                    print("Please enter a value or type 'skip'.")
                    continue

                if val.lower() == "skip":
                    options[opt][criterion] = "unknown"
                    break

                if normalize_key(val) not in [
                    normalize_key(s) for s in scale
                ]:
                    print("Invalid value. Please choose from the scale.")
                    continue

                options[opt][criterion] = val
                break

        else:
            # Numeric value: same idea, explicit skip
            while True:
                val = input(
                    f"  {opt} (enter number{f' ({unit})' if unit else ''} or type 'skip'): "
                ).strip()

                if not val:
                    print("Please enter a number or type 'skip'.")
                    continue

                if val.lower() == "skip":
                    options[opt][criterion] = None
                    break

                try:
                    options[opt][criterion] = float(val)
                    break
                except ValueError:
                    print("Invalid number. Please try again.")

