package com.abdulrauf.myapplication.ui.Register;

import android.content.Intent;
import android.os.Bundle;
import android.text.TextUtils;
import android.widget.Button;
import android.widget.EditText;
import android.widget.RadioGroup;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

import com.abdulrauf.myapplication.MainActivity;
import com.abdulrauf.myapplication.MainActivity2;
import com.abdulrauf.myapplication.R;
import com.abdulrauf.myapplication.ui.login.LoginActivity;
import com.google.firebase.auth.FirebaseAuth;

public class RegisterActivity extends AppCompatActivity {

    private FirebaseAuth mAuth;
    private EditText emailEditText, passwordEditText;
    private Button registerButton;
    private TextView loginLink;
    private RadioGroup roleRadioGroup;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_register);

        mAuth = FirebaseAuth.getInstance();

        emailEditText = findViewById(R.id.emailEditText);
        passwordEditText = findViewById(R.id.passwordEditText);
        registerButton = findViewById(R.id.registerButton);
        loginLink = findViewById(R.id.loginLink);
        roleRadioGroup = findViewById(R.id.roleRadioGroup); // مقداردهی رادیوگروپ

        registerButton.setOnClickListener(v -> register());

        loginLink.setOnClickListener(v -> {
            Intent intent = new Intent(RegisterActivity.this, LoginActivity.class);
            startActivity(intent);
            finish();
        });
    }

    private void register() {
        String email = emailEditText.getText().toString().trim();
        String password = passwordEditText.getText().toString().trim();

        if (TextUtils.isEmpty(email) || TextUtils.isEmpty(password)) {
            Toast.makeText(this, "ایمیل و رمز عبور را وارد کنید", Toast.LENGTH_SHORT).show();
            return;
        }

        int selectedRoleId = roleRadioGroup.getCheckedRadioButtonId();

        if (selectedRoleId == -1) {
            Toast.makeText(this, "لطفاً نقش خود را انتخاب کنید", Toast.LENGTH_SHORT).show();
            return;
        }

        mAuth.createUserWithEmailAndPassword(email, password)
                .addOnSuccessListener(authResult -> {
                    Toast.makeText(this, "ثبت‌نام موفق!", Toast.LENGTH_SHORT).show();

                    Intent intent;
                    if (selectedRoleId == R.id.radioCustomer) {
                        intent = new Intent(RegisterActivity.this, MainActivity.class);
                    } else if (selectedRoleId == R.id.radioBarber) {
                        intent = new Intent(RegisterActivity.this, MainActivity2.class);
                    } else {
                        Toast.makeText(this, "نقش نامعتبر است", Toast.LENGTH_SHORT).show();
                        return;
                    }

                    startActivity(intent);
                    finish();
                })
                .addOnFailureListener(e ->
                        Toast.makeText(this, "خطا در ثبت‌نام: " + e.getMessage(), Toast.LENGTH_SHORT).show());
    }
}
