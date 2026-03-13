---
name: writing-forms
type: pattern
description: Enforces form patterns using React Hook Form and Zod validation. This is reference material — consult during implementation for form conventions.
---

# Writing Forms

Patterns for implementing forms with React Hook Form and Zod validation.

## Standard Form Pattern

```tsx
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import * as z from "zod";

// 1. Define schema — single source of truth for validation
const createAccountSchema = z.object({
    firstName: z.string().min(1, "First name is required"),
    lastName: z.string().min(1, "Last name is required"),
    email: z.string().email("Invalid email address"),
    password: z.string().min(8, "Password must be at least 8 characters"),
});

// 2. Infer type from schema — never define types separately
type CreateAccountData = z.infer<typeof createAccountSchema>;

// 3. Component
const CreateAccountForm = ({ onComplete }: { onComplete: (data: CreateAccountData) => void }) => {
    const {
        register,
        handleSubmit,
        formState: { errors, isSubmitting },
        setError,
    } = useForm<CreateAccountData>({
        resolver: zodResolver(createAccountSchema),
        defaultValues: {
            firstName: "",
            lastName: "",
            email: "",
            password: "",
        },
    });

    const onSubmit = async (data: CreateAccountData) => {
        const response = await submitAccount(data);

        if (response.error) {
            setError("root", { message: response.message });
            return;
        }

        onComplete(data);
    };

    return <form onSubmit={handleSubmit(onSubmit)}>{/* Fields */}</form>;
};
```

## Field Pattern

```tsx
// Complete field: label + input + error
<div className="space-y-2">
    <label htmlFor="email">Email</label>
    <input
        id="email"
        type="email"
        data-testid="create-account-email-input"
        {...register("email")}
        aria-invalid={!!errors.email}
        aria-describedby={errors.email ? "email-error" : undefined}
    />
    {errors.email && (
        <p id="email-error" className="text-sm text-destructive">
            {errors.email.message}
        </p>
    )}
</div>
```

## Multi-Step Form Pattern

For enrollment's multi-step flow (CreateAccount → Products → Shipping → Payment → Agreements).
Use per-step schemas (below) when steps validate independently. Use a shared `FormProvider` with a single `useForm()` when steps share fields or cross-validate (see `managing-state` skill).

```tsx
const EnrollmentForm = ({ steps, flowConfig }: EnrollmentFormProps) => {
    const [currentStep, setCurrentStep] = useState(0);

    // Each step has its own schema, combined for the full form
    const methods = useForm({
        resolver: zodResolver(steps[currentStep].schema),
        mode: "onBlur",
    });

    const handleNext = async () => {
        const isValid = await methods.trigger();
        if (isValid) setCurrentStep((prev) => prev + 1);
    };

    const StepComponent = steps[currentStep].component;

    return (
        <FormProvider {...methods}>
            <StepComponent />
            <StepNavigation
                onNext={handleNext}
                onBack={() => setCurrentStep((prev) => prev - 1)}
                isFirst={currentStep === 0}
                isLast={currentStep === steps.length - 1}
            />
        </FormProvider>
    );
};
```

## Validation Patterns

```tsx
// Country-specific validation
const shippingSchema = (country: string) =>
    z.object({
        address1: z.string().min(1),
        city: z.string().min(1),
        state: country === "US" ? z.string().length(2) : z.string().optional(),
        postalCode: z.string().min(1),
        country: z.string().length(2),
    });

// Conditional fields
const accountSchema = z.discriminatedUnion("accountType", [
    z.object({
        accountType: z.literal("individual"),
        ssn: z.string().optional(),
    }),
    z.object({
        accountType: z.literal("business"),
        taxId: z.string().min(1, "Tax ID required for business"),
    }),
]);
```

## Loading & Error States

```tsx
const SubmitButton = ({ isSubmitting }: { isSubmitting: boolean }) => (
    <button
        type="submit"
        disabled={isSubmitting}
        data-testid="form-submit-button"
    >
        {isSubmitting ? "Submitting..." : "Continue"}
    </button>
);

// Root-level server errors
{errors.root && (
    <div role="alert" data-testid="form-error-message">
        {errors.root.message}
    </div>
)}
```

## Mutation Submit Pattern

Use React Query mutations for form submissions that call the API:

```tsx
import { useSubmitEnrollmentMutation } from "queries/useSubmitEnrollmentMutation";

const EnrollmentForm = () => {
    const { mutateAsync, isPending } = useSubmitEnrollmentMutation();
    const {
        handleSubmit,
        setError,
        formState: { errors },
    } = useForm<EnrollmentData>({
        resolver: zodResolver(enrollmentSchema),
    });

    const onSubmit = async (data: EnrollmentData) => {
        try {
            await mutateAsync(data);
            router.push("/thank-you");
        } catch (error) {
            setError("root", { message: getUserFriendlyMessage(error) });
        }
    };

    return (
        <form onSubmit={handleSubmit(onSubmit)}>
            {/* Fields */}
            {errors.root && (
                <div role="alert" data-testid="form-error-message">
                    {errors.root.message}
                </div>
            )}
            <button type="submit" disabled={isPending}>
                {isPending ? "Submitting..." : "Continue"}
            </button>
        </form>
    );
};
```

See `data-fetching` skill for mutation hook patterns.

## Anti-Patterns

```tsx
// BAD: Manual state for form fields
const [email, setEmail] = useState("");
const [emailError, setEmailError] = useState("");

// GOOD: RHF manages all form state
const { register, formState: { errors } } = useForm();

// BAD: useEffect to validate
useEffect(() => {
    if (!email.includes("@")) setEmailError("Invalid");
}, [email]);

// GOOD: Zod schema validates automatically
const schema = z.object({ email: z.string().email() });

// BAD: Storing form data in context
const { setOrderData } = useOrderContext();

// GOOD: RHF is the single source of truth
const methods = useForm();
// Only extract data on submit, pass to server action
```

## Testing Forms

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

test("shows validation errors on empty submit", async () => {
    render(<CreateAccountForm onComplete={jest.fn()} />);

    await userEvent.click(screen.getByTestId("form-submit-button"));

    expect(screen.getByText("First name is required")).toBeInTheDocument();
    expect(screen.getByText("Invalid email address")).toBeInTheDocument();
});

test("calls onComplete with form data on valid submit", async () => {
    const onComplete = jest.fn();
    render(<CreateAccountForm onComplete={onComplete} />);

    await userEvent.type(screen.getByTestId("create-account-email-input"), "test@example.com");
    // ... fill other fields
    await userEvent.click(screen.getByTestId("form-submit-button"));

    expect(onComplete).toHaveBeenCalledWith(expect.objectContaining({
        email: "test@example.com",
    }));
});
```

## Checklist

- [ ] Schema defined with Zod (single source of truth)
- [ ] Types inferred from schema (`z.infer<typeof schema>`)
- [ ] `useForm` with `zodResolver` — no manual validation
- [ ] `defaultValues` set for all fields
- [ ] Server errors use `setError("root", ...)` or field-specific `setError`
- [ ] Submit button disabled during `isSubmitting`
- [ ] Fields have `aria-invalid` and `aria-describedby` for accessibility
- [ ] All interactive elements have `data-testid`
- [ ] No form state in React context — RHF is the source of truth
- [ ] Multi-step forms use `FormProvider` + per-step schemas
- [ ] API submissions use `useMutation` from React Query (see `data-fetching` skill)
